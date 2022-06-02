# DGL适配教程
> 本教程不介绍DGL是什么，为什么要额外改(如有需要请自行百度)，只介绍怎么改,读者可以clone dgl/hilander项目参照下面修改代码进行理解，注意区分batch_size为1和大于1的情况。

原始代码片段1（GCN代码）：
```python
graph.srcdata['h'] = srcfeat.to('cpu')
a = fn.u_mul_e('h', 'affine', 'm')
b = fn.sum(msg='m', out='h')
graph.update_all(a, b)
gcn_feat = torch.cat([dstfeat, graph.dstdata['h']], dim=-1)
```

修改代码：
```python
gidx = graph._graph
src, dst, edge = gidx.edges(0)
adj = self.build_adj(src, dst, srcfeat)
out = self.gcn(srcfeat.unsqueeze(0).permute(0, 2, 1), adj.to(srcfeat.device)).permute(0, 2, 1)[0, torch.unique(dst).long(), ...]
gcn_feat = torch.cat([dstfeat, out], dim=-1)

@staticmethod
def build_adj(src, dst, srcfeat, directed=True):
   adj = torch.zeros(size=(srcfeat.shape[0], srcfeat.shape[0]))
   for i in range(src.shape[0]):
       adj[src[i]][dst[i]] = 1
   return adj
```

原始代码片段2（forward代码）：
```python
     output_bipartite = bipartites[-1]
     output_bipartite.srcdata['conv_features'] = neighbor_x_src.to('cpu')
     output_bipartite.dstdata['conv_features'] = center_x_src.to('cpu')

 output_bipartite.apply_edges(self.pred_conn)
 output_bipartite.edata['prob_conn'] = F.softmax(output_bipartite.edata['pred_conn'], dim=1)
 output_bipartite.update_all(self.pred_den_msg, fn.mean('pred_den_msg', 'pred_den'))
 return output_bipartite
```

修改代码：
```python
    output_bipartite = bipartites[-1]
    # if self.raw_affine is None:
    #     self.raw_affine = nn.Parameter(output_bipartite.edata['raw_affine'].to(neighbor_x_src.device))
# print("forward with raw_affine shape=", output_bipartite.edata['raw_affine'].shape, ", data=", output_bipartite.edata['raw_affine'][:9])
gidx = output_bipartite._graph
src, dst, edge = gidx.edges(0)
# 多batch下取出目标节点还有点问题估计
src_feat = self.src_mlp(neighbor_x_src[src.long()])
###  feature alignment
len_center_x_src = center_x_src.shape[0]
len_src_feat = src_feat.shape[0]
repeat = int(len_src_feat / len_center_x_src)
center_x_src_r = torch.repeat_interleave(center_x_src, repeat, dim=0)
###
dst_feat = self.dst_mlp(center_x_src_r)
pred_conn = self.classifier_conn(src_feat + dst_feat)
output_bipartite.edata['pred_conn'] = pred_conn.to('cpu')  # 保存用于计算loss
prob = F.softmax(pred_conn, dim=1)
raw_affine = output_bipartite.edata['raw_affine'].to(device)
pred_den_msg = raw_affine * (prob[:, 1] - prob[:, 0])
### 分组mean
g_pred_den_msg = torch.cat([pred_den_msg[i:i + repeat].unsqueeze(0) for i in range(0, len(pred_den_msg), repeat)], dim=0)
###
pred_den = torch.mean(g_pred_den_msg, dim=1)
output_bipartite.dstdata['pred_den'] = pred_den.to('cpu')  # 存到cpu上中转，不会丢失梯度
return output_bipartite
```

原始代码片段3（Loss代码）：
```python
# 参考原始文件
```

修改代码：
```python
def compute_loss(self, bipartite, device='cpu'):
    pred_den = bipartite.dstdata['pred_den'].to(device)
    loss_den = self.loss_den(pred_den, bipartite.dstdata['density'].to(device))  # MSE-Loss

    labels_conn = bipartite.edata['labels_conn'].int().to(device)
    mask_conn = bipartite.edata['mask_conn'].to(device)

    if self.balance:
        # labels_conn = bipartite.edata['labels_conn']
        neg_check = torch.logical_and(labels_conn == 0, mask_conn)
        num_neg = torch.sum(neg_check).item()
        neg_indices = torch.where(neg_check)[0]
        pos_check = torch.logical_and(labels_conn == 1, mask_conn)
        num_pos = torch.sum(pos_check).item()
        pos_indices = torch.where(pos_check)[0]
        if num_pos > num_neg:
            mask_conn[pos_indices[np.random.choice(num_pos, num_pos - num_neg, replace = False)]] = 0
        elif num_pos < num_neg:
            mask_conn[neg_indices[np.random.choice(num_neg, num_neg - num_pos, replace = False)]] = 0

    # In subgraph training, it may happen that all edges are masked in a batch
    if mask_conn.sum() > 0:
        loss_conn = self.loss_conn(bipartite.edata['pred_conn'].to(device)[mask_conn], labels_conn[mask_conn].long())
        loss = loss_den + loss_conn
        loss_den_val = loss_den.item()
        loss_conn_val = loss_conn.item()
    else:
        loss = loss_den
        loss_den_val = loss_den.item()
        loss_conn_val = 0

    return loss, loss_den_val, loss_conn_val
```
