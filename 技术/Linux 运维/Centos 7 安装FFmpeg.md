使用Nux Dextop存储库中的yum进行安装

Nux资料库取决于[ EPEL ](https://www.myfreax.com/how-to-enable-epel-repository-on-centos/)软件资料库。启用EPEL

```bash
yum install epel-release
```

导入存储库GPG密钥并通过[安装rpm软件包来启用Nux存储库](https://www.myfreax.com/how-to-install-rpm-packages-on-centos/)

```bash
rpm -v --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
```

安装FFmpeg

```bash
yum install ffmpeg ffmpeg-devel
```

# 参考资料

[如何在CentOS 7上安装和使用FFmpeg](https://www.myfreax.com/how-to-install-ffmpeg-on-centos-7/)