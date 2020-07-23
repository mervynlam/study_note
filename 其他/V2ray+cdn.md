[TOC]

# V2ray + cdn + ws +tls

[参考文章](https://umrhe.com/v2ray-nginx-ws-tls-cdn-to-google.html)

## Freenom免费域名 + Cloudflare + BNXB

### Freenom管理域名（建议挂梯子）

[Freenom官网](https://www.freenom.com/zh/index.html?lang=zh)

1. 购买域名（不附图说明）
   1. 输入想要的域名检查是否可用
   2. 获取域名，若出现不可用，直接在搜索时同时输入想要的后缀，如`Mervyn.tk`
   3. 选择期限，免费最长12个月
   4. 输入邮箱验证
   5. 邮箱打开网站输入本人正确信息，如果挂了梯子，需要使用pac模式，或是选择与梯子相同的城市，Freenom会检测ip所在地
2. 管理域名
   1. My Domains 管理所有域名
   2. Renew Domains 更新域名有效期

### Cloudflare + BNXB

[参考文章](https://zhang.ge/5149.html)

1. 注册Cloudflare

   [Cloudflare官网](https://www.cloudflare.com/)

2. 注册BNXB

   [BNXB官网](https://cdn.bnxb.com/)

   1. 添加域名

      ![image-20200216133736936](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200216133736936.png)

   2. 解析设置

      ![image-20200215221218281](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200215221218281.png)

   3. 添加解析记录

      ![image-20200216133854773](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200216133854773.png)

   4. 到域名DNS添加CNAME解析（DNSPOD）

      ![image-20200216133822956](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200216133822956.png)

      ![image-20200216133928324](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200216133928324.png)

## 实现V2ray + cdn

1. 安装宝塔面板

   ```bash
   curl -sSO http://download.bt.cn/install/new_install.sh && bash new_install.sh
   ```

   记录地址、账号、密码

   ![image-20200216123942955](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200216123942955.png)

2. 登录宝塔面板，安装nginx

   ![image-20200216124008881](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200216124008881.png)

3. 添加域名

   ![image-20200216124856617](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200216124856617.png)

4. 安装v2ray

   ```bash
   wget https://install.direct/go.sh
   bash ./go.sh
   ```

   记录端口和UUID

   ![image-20200216124933876](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200216124933876.png)

   后续可以使用以下命令查看

   ```bash
   cat /etc/v2ray/config.json
   ```

5. 配置SSL

   ![image-20200216125440753](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200216125440753.png)

   若卡在“正在申请证书....”

   ssh执行命令`pip install Pyopenssl`

6. 修改配置文件

   ![image-20200216125624696](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200216125624696.png)

   ```bash
       location /v2ray
       {
           proxy_pass http://127.0.0.1:v2ray的端口号;
           proxy_redirect off;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
           proxy_set_header Host $http_host;
           proxy_read_timeout 300s;
       }
   ```

   第一行的`/v2ray`是ws的路径，v2ray端口号修改为刚刚记录的端口号

7. 配置v2ray的配置文件

   修改`/etc/v2ray/config.json`文件

   ```json
   {
     "policy": {
       "levels": {
         "0": {
           "uplinkOnly": 0,
           "downlinkOnly": 0,
           "connIdle": 150,
           "handshake": 4
         }
       }
     },
     "inbound": {
       "listen": "127.0.0.1",
       "port": 36397,    //这里填写你的 v2ray 端口号，复制脚本请产出这句注释
       "protocol": "vmess",
       "settings": {
         "clients": [
           {
             "id": "ef1563fc-d2a9-4b52-b752-a9c318bf2642",    //这里填写你的 v2ray UUID，复制脚本请删除这句注释
             "level": 1,
             "alterId": 32
           }
         ]
       },
       "streamSettings": {
         "network": "ws",
         "security": "auto",
         "wsSettings": {
           "path": "/v2ray",   //这里填是你自己 ws 的 path,如果修改配置文件的时候没有修改过就不管，复制脚本请删除这句注释
           "headers": {
             "Host": "www.xxx.com"  //这里填写你的域名，复制脚本请删除这句注释
           }
         }
       }
     },
     "outbound": {
       "protocol": "freedom",
       "settings": { }
     },
     "outboundDetour": [
       {
         "protocol": "blackhole",
         "settings": { },
         "tag": "blocked"
       }
     ],
     "routing": {
       "strategy": "rules",
       "settings": {
         "rules": [
           {
             "type": "field",
             "ip": [
               "0.0.0.0/8",
               "10.0.0.0/8",
               "100.64.0.0/10",
               "127.0.0.0/8",
               "169.254.0.0/16",
               "172.16.0.0/12",
               "192.0.0.0/24",
               "192.0.2.0/24",
               "192.168.0.0/16",
               "198.18.0.0/15",
               "198.51.100.0/24",
               "203.0.113.0/24",
               "::1/128",
               "fc00::/7",
               "fe80::/10"
             ],
             "outboundTag": "blocked"
           }
         ]
       }
     }
   }
   
   ```

8. 开放端口

   ```bash
   #centos 开端口的方法
   firewall-cmd --zone=public --add-port=36397/tcp --permanent
   firewall-cmd --reload
   ## 查看已开放端口
   firewall-cmd --zone=public --list-ports
   ```

10. 启动v2ray

    ```bash
    systemctl start v2ray
    #重启
    systemctl restart v2ray
    #开机自启
    systemctl enable v2ray
    ```

11. 配置客户端

     ![image-20200215224212061](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200215224212061.png)

     ![image-20200216133517631](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200216133517631.png)
