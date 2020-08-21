# 启动FTP服务
1. 控制面板->程序->启用或关闭Windows功能
![8638dacc6c3d787668ec2071f8efa4fc.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821112646.png)
2. 小娜搜索IIS打开IIS（或此电脑->管理->服务与应用程序->IIS）
![c9961091bea2e0b296872413e94647da.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821112718.png)
![726843ba610d9f8cb9c8ef60c491680b.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821112743.png)
3. 右击网站添加FTP站点
![b69e195d0f60df36e13a2087f509a5e2.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821112803.png)
4. 输入站点名称和作为FTP的目录
![1955f1a086b5de121e5957a75a84224c.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821112820.png)
5. IP地址填FTP本机IP，选择无SSL
![f6c02cdb78df2c843d87897b9dc43a30.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821113901.png)
6. 根据需要选择身份验证，授权与权限
![41b9b4ed889b23cc2e0e2baa436b9cee.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821113847.png)
7. 控制面板->系统和安全->防火墙->允许应用或功能通过Window防火墙
![314c6f8039f6dcba203bfa6c92a23e47.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821113931.png)
8. 点击更改设置，勾选FTP服务器
![dda893149f6b5acc6cc1929750f110b3.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821113953.png)
9. 点击允许其它应用,选择C:\Windows\System32\svchost.exe然后添加，最后确定
![26e242d3c05cecc7c461e679ac9551dc.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821114009.png)
10. 资源处理器或浏览器访问`ftp://IP`
# 配置FTP用户名和密码
1. 打开此电脑->管理->本地用户和组->用户
![c5e5b0c4968d73bd99d74637cc2b5c6f.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821122241.png)
2. 建立新用户
![db546afc7a86b65f37d81aa645d733bb.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821122238.png)
![d7bbdabbb3bbdab8bf1d0c512d45f519.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821122235.png)
3. FTP设置登录验证
    小娜搜索IIS打开IIS（或此电脑->管理->服务与应用程序->IIS）
    ![e413b0750699c659c9f9dbbbd35b9349.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821122231.png)
4. FTP身份验证，禁用匿名
![d539c08ccf4befcf9c84e55334f21c18.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821122227.png)
5. FTP授权规则，添加允许规则
![84b73b9ee0589671e5abbe00573e68ce.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821122224.png)
删除允许的所有用户
![4063e0f36d6618142b488bb8367dd477.png](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821122219.png)

# 参考资料
[Win10开启FTP与配置（完整无错版）- zbored](https://blog.csdn.net/qq_34610293/article/details/79210539)
[Win10搭建ftp（含设置用户名和密码） - redhatlxs](https://blog.csdn.net/liu1123055728/article/details/94627962)