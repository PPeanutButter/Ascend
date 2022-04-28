# 使用华为昇腾卡训练
> 为了减少课题组后期重复工作，请大家star这个项目，如果有教程补充的提交pull request，如果有任何项目相关问题请提交issue到官方仓库，解决了的issue再提交到此处留档，方便后期研究。

## 准备数据到OBS
> 不需要直接将数据上传到服务器

1. 将数据集压缩包上传至OBS桶的`dataset`文件夹内。![](npu/1.png)
2. 在本地解压后，将数据集上传至`dataset_unzip`文件夹内。![](npu/2.png)
3. 从解压的数据集中只保留数十张，将数据集上传至`dataset_unzip_mini`文件夹内，用于快速调试。![](npu/3.png)
4. 项目中有任何大型静态文件的，将文件各上传至`dataset_unzip`、`dataset_unzip_mini`, 如上图。

## OBS设置桶公开策略

1、登录华为云后，进入控制台，选择 `北京四`。然后点击 `对象存储服务OBS`
![输入图片说明](npu/obs1.png)
2、进入需要公开读的obs桶 
![输入图片说明](npu/obs1.png)
3、找到访问 `权限控制 - 桶策略`。然后点击 创建
![输入图片说明](npu/obs1.png)
4、在公共读写行，点击使用模板创建。
![输入图片说明](npu/obs1.png)
5、最后点击右下角的 `配置确认` ，然后进入第二个页面点击 `创建`。

## 使用pycharm创建modelarts训练任务
>由于AI开发者会使用PyCharm工具开发算法或模型，为方便快速将本地代码提交到ModelArts的训练环境，ModelArts提供了一个PyCharm插件工具PyCharm ToolKit（[插件下载](https://modelarts-pycharm-plugin.obs.cn-north-1.myhuaweicloud.com/Pycharm-ToolKit-latest.zip)），协助用户完成代码上传、提交训练作业、将训练日志获取到本地展示等，用户只需要专注于本地的代码开发即可。

### 在PyCharm中安装ToolKit工具
> 请根据如下操作指导，将下载的ToolKit工具安装至PyCharm中。

1. 打开本地PyCharm工具。
2. 在PyCharm工具中，选择菜单栏的“File > Settings”，弹出“Settings”对话框。
3. 在“Settings”对话框中，首先单击左侧导航栏中的“Plugins”，然后单击右侧的设置图标，选择“Install Plugin from Disk”，弹出文件选择对话框。
![图1 选择从本地安装插件](npu/toolkit1.png)
4. 在弹出的对话框中，从本地目录选择ToolKit的工具zip包，然后单击“OK”。
![图2 选择插件文件](npu/toolkit2.png)
![5. 单击“Restart IDE”重启PyCharm。在弹出的确认对话框中，单击“Restart”开始重启。
图3 重启PyCharm](npu/toolkit3.png)
6. 重启成功后，打开一个Project，当PyCharm工具栏出现“ModelArts”页签，表示ToolKit工具已安装完成。
![图4 安装成功](npu/toolkit4.png)

### 创建访问密钥（AK和SK）
> 见之前的pdf步骤三或步骤四。

### 登录ModelArts
1. 打开已安装ToolKit工具的PyCharm，在菜单栏中选择“ModelArts > Edit Credential”。
![图1 Edit Credential](npu/toolkit5.png)
2. 在弹出的对话框中，选择您使用的ModelArts所在区域、填写AK、SK，然后单击“OK”完成登录。
    - “Region”：从下拉框中选择区域，目前支持“华北-北京四”、“华北-北京一”和“华东-上海一”区域，必须与ModelArts管理控制台在同一区域。
    - “Access Key ID”：填写访问密钥的AK。
    - “Secret Access Key”：填写访问密钥的SK。
![图2 填写区域和访问密钥](npu/toolkit6.png)
3. 查看认证结果。在Event Log区域中，当提示如下类似信息时，表示访问密钥添加成功。
`16:01Validate Credential Success: The HUAWEI CLOUD credential is valid`.