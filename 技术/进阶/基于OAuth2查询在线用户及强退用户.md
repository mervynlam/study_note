# 基于OAuth2查询在线用户及强退用户

## 场景

最近工作的项目需要做认证授权、限制授权登录用户。

需要统计当前在线用户数，及强退指定用户。

## 效果

![image-20210810094001336](https://raw.githubusercontent.com/mervynlam/Pictures/master/20210810094116.png)

## 实现

**获取所有在线用户**

登录后，`OAuth2`会向`Redis`存放用户的`token`信息，键是`token`值，值是`OAuth2`的认证对象。

实现思路：从`Redis`中获取所有的`token`值，及通过`RedisTokenStore`获取到`OAuth2`的认证对象。

```java
/**
 * VO
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class OnlineUserVo {
    private String token;
    private Long userId;
    private String name;
    private Date loginTime;         //登陆时间
    private Date expiration;        //到期时间
    private Integer expiresIn;      //剩余时间
}

```

```java
/**
 * RedisTokenStore注入配置
 */
@Configuration
@EnableAuthorizationServer
public class TokenStoreConfig {
    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Bean
    public RedisTokenStore redisTokenStore() {
        return new RedisTokenStore(redisConnectionFactory);
    }
}
```

```java
//redis存放token的键前缀，可以从RedisTokenStore源码中看到
public static final String REDIS_AUTH_KEY="auth:";

@Autowired
private RedisTokenStore redisTokenStore;

public List<OnlineUserVo> getOnlineUserList() {
    List<OnlineUserVo> list = new ArrayList<>();
    //oauth存放的所有keys
    // Set<String> keys = redisTemplate.keys(OnlineConstants.REDIS_AUTH_KEY + "*");
    //经elliotxiang提醒，keys会影响redis使用，修改为scan
    Set<String> keys = scan(REDIS_AUTH_KEY);
    for (String key : keys) {
        //keys = "auth:" + token
        //根据token获取OAuth2AccessToken
        String token = key.substring(key.indexOf(":") + 1);
        //根据token值获取OAuth2AccessToken对象
        OAuth2AccessToken oAuth2AccessToken = redisTokenStore.readAccessToken(token);
        
        //获取用户id，本项目用户id存放在OAuth2AccessToken对象中的additionalInformation属性里。
        //根据实际情况修改
        Map<String, Object> map = oAuth2AccessToken.getAdditionalInformation();
        if (map != null) {
            //获取用户id
            Long userId = (Long) map.get("openid");
            if (userId != null) {
                //获取用户信息
                BaseUser user = this.userService.getUserById(userId);
                //获取用户登录日志
                BaseAccountLogs logs = this.accountLogsService.getUserNewestLogin(userId);

                list.add(new OnlineUserVo(token, userId, user.getFullName(),
                                          logs.getLoginTime(),
                                          oAuth2AccessToken.getExpiration(),
                                          oAuth2AccessToken.getExpiresIn()));
            }
        }
    }
    return list;
}

/
public Set<String> scan(String matchKey) {
    Set<String> keys = (Set<String>) redisTemplate.execute((RedisCallback<Set<String>>) connection -> {
        Set<String> keysTmp = new HashSet<>();
        Cursor<byte[]> cursor = connection.scan(new ScanOptions.ScanOptionsBuilder().match(matchKey + "*").count(1000).build());
        while (cursor.hasNext()) {
            keysTmp.add(new String(cursor.next()));
        }
        return keysTmp;
    });

    return keys;
}
```

**获取在线用户数**

获取在线用户数就简单许多，只需要获取`Redis`中`token`键的数量即可。

```java
public Integer getOnlineUserNum() {
    //Set<String> keys = redisTemplate.keys(OnlineConstants.REDIS_AUTH_KEY + "*");
    //经elliotxiang提醒，keys会影响redis使用，修改为scan
    Set<String> keys = scan(REDIS_AUTH_KEY);
    return keys.size();
}
```

**强制登出用户**

将用户`token`从`Redis`中删除即可

```java
public void forceLogout(String token) {
    redisTokenStore.removeAccessToken(token);
}
```

## 参考资料

[若依管理系统](http://vue.ruoyi.vip/login?redirect=%2Fmonitor%2Fonline)

