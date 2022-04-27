# 使用华为昇腾卡训练
> 为了减少课题组后期重复工作，请大家star这个项目，如果有教程补充的提交pull request，如果有任何项目相关问题请提交issue到官方仓库，解决了的issue再提交到此处留档，方便后期研究。

## 准备数据到OBS
> 不需要直接将数据上传到服务器

1. 将数据集压缩包上传至OBS桶的`dataset`文件夹内。![](npu/1.png)
2. 在本地解压后，将数据集上传至`dataset_unzip`文件夹内。![](npu/2.png)
3. 从解压的数据集中只保留数十张，将数据集上传至`dataset_unzip_mini`文件夹内，用于快速调试。![](npu/3.png)
4. 项目中有任何大型静态文件的，将文件各上传至`dataset_unzip`、`dataset_unzip_mini`, 如上图。

## 使用pycharm创建modelarts训练任务