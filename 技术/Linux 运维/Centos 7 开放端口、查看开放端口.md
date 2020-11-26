## 开放端口

```bash
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload
```

**命令含义**

> --zone #作用域
> --add-port=80/tcp #添加端口，格式为：端口/通讯协议
> --permanent #永久生效，没有此参数重启后失效

## 关闭端口

```bash
firewall-cmd --zone=public --remove-port=80/tcp --permanent
firewall-cmd --reload
```

## 查看开放端口

```bash
firewall-cmd --list-ports
```

