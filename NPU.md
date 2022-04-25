# 使用华为昇腾卡训练
> 为了减少课题组后期重复工作，请大家star这个项目，如果有教程补充的提交pull request，如果有任何项目相关问题请提交issue留档，方便后期研究。

## 购买卡
> 前期建议购买最便宜的310[点我购买](https://console.huaweicloud.com/ecm/?region=cn-north-4&locale=zh-cn#/ecs/createVm?charging=0&az=cn-north-4b&flavor=ai1.large.4&imageId=07647e58-b16a-4982-aab1-1814213f833f&vpcid=265663cf-f658-4caa-82b8-0c154c170db1&server_id=e74aa64e-2f90-4caf-8e3d-c1b2044de605&sgs=0e86563b-5ae8-4d9b-bcd5-aa2642910c0e&nics=5afbd3ec-d95f-4316-aef0-d6e6918904e2&sysdisk=GPSSD:100:0:1&datadisk=&isDssSysDisk=false&buySameConfECS=1&iptype=5_bgp&ipcharging=traffic&bwsize=300)卡用于调试，确保能跑后迁移到910上。

规格按照`AI加速型 | ai1.large.4 | 2vCPUs | 8GiB | 1 * HUAWEI Ascend 310/1 * 8G`购买即可，硬盘、网络按需添加。

## 准备数据到服务器
> 不需要直接将数据上传到服务器

1. 将数据上传至OBS桶。
2. 将OBS内的数据集复制到服务器上（或者前期你单独新建了数据盘的话，可以直接用），执行`wget <从obs内获取分享的链接>`，~200MB/s。
3. 解压数据集，一般数据集上传都是压缩包。
```shell
.zip 使用 unzip <file>
.tar.gz 使用 tar -zxvf <file> 
```
