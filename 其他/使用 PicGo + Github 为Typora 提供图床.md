# 使用 PicGo + Github 为Typora 提供图床

Typora 是一款非常好用的跨平台“所见即所得”的 Markdown 编辑器。一直使用 Typora 编写 Markdown 文本：博客、笔记之类的。

但是在写笔记、博客时，使用的图片都是本地路径，上传到 Github 后，会有图片无法显示的问题。

所以今天学习一下使用 PicGO + Github 建立自己的图床。

## 软件下载

[Typora](http://typora.io/)

[PicGo](https://github.com/Molunerfinn/PicGo/releases)

## 创建图片仓库

Github 中创建一个用于存放图片的库。

## 创建一个Token 供 PicGo使用

访问 [https://github.com/settings/tokens](https://github.com/settings/tokens)

点击 `Generate new token`

![image-20200821104929598](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821104929.png)

填写标签，勾选 repo ，然后确认创建

![image-20200821105039323](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821105039.png)

点击复制备份，后续需要使用

![image-20200821105205642](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821105205.png)

## 配置 PicGo

1. 进入 Github 图床 设置，仓库名输入`用户名/仓库`，分支一般为`master`即可，将刚刚获取到的 token 填入。点击确认，并设为默认图床。

   ![image-20200821105315490](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821105315.png)

2. 开启 时间戳重命名，避免图片名重复造成错误

   ![image-20200821105753216](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821105753.png)

## 配置 Typora

1. 插入图片即上传图片，针对本地或网络图片都应用这个规则。

2. 上传服务选PicGo(app)，选择安装路径。

3. 确认无误后，点击 验证图片上传选项。

   ![image-20200821105642202](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821105642.png)

4. 如此即为配置成功

   ![image-20200821110050350](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821110050.png)

# 参考资料

[PicGo 指南](https://picgo.github.io/PicGo-Doc/zh/guide/)

