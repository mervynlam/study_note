# Java 实现 RSA 非对称加密

## RSA 加解密

RSA是一种非对称加密算法，拥有一对密钥（公钥和私钥），由公钥加密的数据只能由私钥解密，由私钥加密的数据只能由公钥解密。

### RSA 工具类

```java
public class RsaUtils {
    //加密算法RSA
    private static final String ALGORITHM = "RSA";
    //密钥长度
    private static final int KEY_SIZE = 1024;
    //公钥文件
    private static final String PUBLIC_KEY_FILE = "PublicKey";
    //密钥文件
    private static final String PRIVATE_KEY_FILE = "PrivateKey";

    /**
     * @Title: generateKeyPair
     * @Description: TODO(生成并保存密钥对)
     * @author mervyn
     * @date  2020年05月21日 10:59:07
     * @param
     * @return void
     * @throws
    */
    public static void generateKeyPair() {
        //获取一个RSA算法的密钥生成实例
        KeyPairGenerator keyPairGenerator = null;
        try {
            keyPairGenerator = KeyPairGenerator.getInstance(ALGORITHM);
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }

        //初始化密钥长度
        keyPairGenerator.initialize(KEY_SIZE);

        //生成密钥对
        KeyPair keyPair = keyPairGenerator.generateKeyPair();

        //公钥
        RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
        //私钥
        RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();

        //编码处理，方便保存
        String publicKeyStr = new BASE64Encoder().encode(publicKey.getEncoded());
        String privateKeyStr = new BASE64Encoder().encode(privateKey.getEncoded());

        //保存输出流
        BufferedWriter pubBw = null;
        BufferedWriter priBw = null;
        try {
            pubBw = new BufferedWriter(new FileWriter(PUBLIC_KEY_FILE));
            priBw = new BufferedWriter(new FileWriter(PRIVATE_KEY_FILE));
            pubBw.write(publicKeyStr);
            priBw.write(privateKeyStr);
            pubBw.flush();
            priBw.flush();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (pubBw != null) {
                try {
                    pubBw.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (priBw != null) {
                try {
                    priBw.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * @Title: getPublicKey
     * @Description: TODO(获取公钥)
     * @author mervyn
     * @date  2020年05月21日 11:27:49
     * @param
     * @return java.security.interfaces.RSAPublicKey
     * @throws
    */
    private static RSAPublicKey getPublicKey() {
        //输入流
        BufferedReader pubBr = null;
        RSAPublicKey publicKey = null;
        try {
            pubBr = new BufferedReader(new FileReader(PUBLIC_KEY_FILE));
            String line = null;
            StringBuilder publicKeyEncode = new StringBuilder();
            while ((line = pubBr.readLine()) != null) {
                publicKeyEncode.append(line);
            }

            //base64转为byte
            byte[] publicKeyByte = new BASE64Decoder().decodeBuffer(publicKeyEncode.toString());
            //获取指定算法的密钥工厂, 根据 已编码的公钥规格, 生成公钥对象
            X509EncodedKeySpec encodedKeySpec = new X509EncodedKeySpec(publicKeyByte);
            publicKey = (RSAPublicKey) KeyFactory.getInstance(ALGORITHM).generatePublic(encodedKeySpec);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (pubBr != null) {
                try {
                    pubBr.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return publicKey;
    }

    /**
     * @Title: getPrivateKey
     * @Description: TODO(获取私钥)
     * @author mervyn
     * @date  2020年05月21日 13:56:34
     * @param
     * @return java.security.interfaces.RSAPrivateKey
     * @throws
    */
    private static RSAPrivateKey getPrivateKey() {
        //输入流
        BufferedReader priBr = null;
        RSAPrivateKey privateKey = null;
        try {
            priBr = new BufferedReader(new FileReader(PRIVATE_KEY_FILE));
            String line = null;
            StringBuilder privateKeyEncode = new StringBuilder();
            while ((line = priBr.readLine()) != null) {
                privateKeyEncode.append(line);
            }

            //base64转为byte
            byte[] privateKeyByte = new BASE64Decoder().decodeBuffer(privateKeyEncode.toString());
            //获取指定算法的密钥工厂, 根据 已编码的私钥规格, 生成私钥对象
            PKCS8EncodedKeySpec encodedKeySpec = new PKCS8EncodedKeySpec(privateKeyByte);
            privateKey = (RSAPrivateKey) KeyFactory.getInstance(ALGORITHM).generatePrivate(encodedKeySpec);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (priBr != null) {
                try {
                    priBr.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return privateKey;
    }

    /**
     * @Title: encrypt
     * @Description: TODO(公钥加密)
     * @author mervyn
     * @date  2020年05月21日 14:00:56
     * @param sourceStr
     * @return java.lang.String
     * @throws
    */
    public static String encrypt(String sourceStr) {
        //获取公钥
        RSAPublicKey publicKey = getPublicKey();
        String encodeStr = null;
        try {
            //RSA加密
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.ENCRYPT_MODE, publicKey);
            encodeStr = new BASE64Encoder().encode(cipher.doFinal(sourceStr.getBytes()));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return encodeStr;
    }

    /**
     * @Title: decrypt
     * @Description: TODO(私钥解密)
     * @author mervyn
     * @date  2020年05月21日 14:05:53
     * @param encodeStr
     * @return java.lang.String
     * @throws
    */
    public static String decrypt(String encodeStr) {
        //获取私钥
        RSAPrivateKey privateKey = getPrivateKey();
        String sourceStr = null;
        try {
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.DECRYPT_MODE, privateKey);
            sourceStr = new String(cipher.doFinal(new BASE64Decoder().decodeBuffer(encodeStr)));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return sourceStr;
    }
}
```

### 调用及结果

```java
public static void main(String[] args) {
    RsaUtils.generateKeyPair();
    String source = "RSA非对称加解密DEMO";
    System.out.println("源文: " + source);
    String encryptStr = RsaUtils.encrypt(source);
    System.out.println("加密: " + encryptStr);
    String decryptStr = RsaUtils.decrypt(encryptStr);
    System.out.println("解密: " + decryptStr);
}
```

```
源文: RSA非对称加解密DEMO
加密: moyPJXwzpeRsq3OzaEesILDQ9L23+wqCe+FOztIkXuEYkt+aWdsZjrP+cFi4lasVf3iBXjrQWR1H
DfAavszchn8FUmFveZvwgBPi5E9lcdvWN+uuPiA9UkkisMvV+dtRn1eBnpW6XvoEQ1CNahI+ZKst
+pQGf8Gqer0UWFCzVHA=
解密: RSA非对称加解密DEMO
```

## RSA 分段加解密

RSA 加解密有字节长度限制

- 加密长度：密钥长度 / 8 -11。2048位的证书，加密时最大支持245个字节
- 解密长度：密钥长度 / 8。2048位的证书，解密时最大支持256个字节

### 分段加解密代码

```java
//最大加密长度
private static final int MAX_ENCRYPT_BLOCK = KEY_SIZE / 8 -11;
//最大解密长度
private static final int MAX_DECRYPT_BLOCK = KEY_SIZE / 8;
/**
* @Title: encryptSegment
* @Description: TODO(分段加密)
* @author mervyn
* @date  2020年05月21日 16:07:04
* @param sourceStr
* @return java.lang.String
* @throws
*/
public static String encryptSegment(String sourceStr) {
    //获取公钥
    RSAPublicKey publicKey = getPublicKey();
    String encryptStr = null;
    try {
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.ENCRYPT_MODE, publicKey);
        //获取源文字节
        byte[] sourceByte = sourceStr.getBytes();
        //源文长度
        int sourceLen = sourceByte.length;
        //偏离度
        int offset = 0;
        //分段次数
        int tmp = 0;
        //分段缓存
        byte[] cache;
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        //循环字节
        while (sourceLen - offset > 0) {
            if (sourceLen - offset > MAX_ENCRYPT_BLOCK) {
                cache = cipher.doFinal(sourceByte, offset, MAX_ENCRYPT_BLOCK);
            } else {
                cache = cipher.doFinal(sourceByte, offset, sourceLen - offset);
            }
            out.write(cache, 0, cache.length);
            tmp++;
            offset = tmp * MAX_ENCRYPT_BLOCK;
        }
        byte[] result = out.toByteArray();
        out.close();
        encryptStr = new BASE64Encoder().encode(result);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return encryptStr;
}

/**
* @Title: decryptSegment
* @Description: TODO(分段解密)
* @author mervyn
* @date  2020年05月21日 16:34:34
* @param encryptStr
* @return java.lang.String
* @throws
*/
public static String decryptSegment (String encryptStr) {
    //获取私钥
    RSAPrivateKey privateKey = getPrivateKey();
    String sourceStr = null;
    try {
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.DECRYPT_MODE, privateKey);
        //获取加密文字节
        byte[] encryptByte = new BASE64Decoder().decodeBuffer(encryptStr);
        //加密文长度
        int encryptLen = encryptByte.length;
        //偏离度
        int offset = 0;
        //分段次数
        int tmp = 0;
        //分段缓存
        byte[] cache;
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        //循环字节
        while (encryptLen - offset > 0) {
            if (encryptLen - offset > MAX_DECRYPT_BLOCK) {
                cache = cipher.doFinal(encryptByte, offset, MAX_DECRYPT_BLOCK);
            } else {
                cache = cipher.doFinal(encryptByte, offset, encryptLen - offset);
            }
            out.write(cache, 0, cache.length);
            tmp++;
            offset = tmp * MAX_DECRYPT_BLOCK;
        }
        byte[] result = out.toByteArray();
        out.close();
        sourceStr = new String(result);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return sourceStr;
}
```

### 调用及结果

```java
public static void main(String[] args) {
    RsaUtils.generateKeyPair();
    String source = "既然如此， 了解清楚科学和人文谁更有意义到底是一种怎么样的存在，是解决一切问题的关键。 奥普拉·温弗瑞曾经提到过，你相信什么，你就成为什么样的人。这句话语虽然很短，但令我浮想联翩。 伏尔泰曾经说过，坚持意志伟大的事业需要始终不渝的精神。这不禁令我深思。 要想清楚，科学和人文谁更有意义，到底是一种怎么样的存在。 一般来讲，我们都必须务必慎重的考虑考虑。 我们不得不面对一个非常尴尬的事实，那就是， 这样看来。";
    System.out.println("源文: " + source);
    String encryptStr = RsaUtils.encryptSegment(source);
    System.out.println("加密: " + encryptStr);
    String decryptStr = RsaUtils.decryptSegment(encryptStr);
    System.out.println("解密: " + decryptStr);
}
```

```
源文: 既然如此， 了解清楚科学和人文谁更有意义到底是一种怎么样的存在，是解决一切问题的关键。 奥普拉·温弗瑞曾经提到过，你相信什么，你就成为什么样的人。这句话语虽然很短，但令我浮想联翩。 伏尔泰曾经说过，坚持意志伟大的事业需要始终不渝的精神。这不禁令我深思。 要想清楚，科学和人文谁更有意义，到底是一种怎么样的存在。 一般来讲，我们都必须务必慎重的考虑考虑。 我们不得不面对一个非常尴尬的事实，那就是， 这样看来。
加密: jy15NKLCixEsHtTO1civuIPEGqjhIqjM3AaOt/GGwQMNGuEnd+LbD/qFeLaiONi3OVxy9NVQXziR
T1l4TrPIvb33kxCdFpwWv0mvTZwnHCloQpmuLRGr4LFH1/7H6/WIYK+NK82S74sH0HiHMUiTpoVp
TQsoJYA8NRyFpbNlIwIMFKSkkUJdBtDBFktFkNmgOzMKBBQwo2q/r3RG83rCW9IByrrxX9KtBAXF
stp1lE8fsKGhjrNbAkb3Ouy08i5Lh9oDbpkai/YBV3otGU2Yx3EGZM7YKOw7EvzknirMBo6qt0Q/
VbKnVD4npQ7FywhyTY668l9EsTPvBDyO6zDgZn6PhVroxDYIc/PCbakK4y1XGgew1ZvFq/I7NA3P
rIkgj1ICzH+XO6oc/xgK7jlEhcXPolngNI3AdUjUmI8qmfpQzMuYBoB3lX38GmhjWONZkxASu1S3
K+4XmSGB2a5o/fJxCV4jsqUAHhjua9dSN5A2Kdtnzm8MmyUPzwaCz+qNCmwpNEe6Sqvl/Nm4hgBm
8tHRt/vKsgh7oyrt3Y2MVf9lvomcIeEtNnZfd19KrOkMuj2vYIDvjLwbNAB/K+tC5Q5OuxHS3IBn
KnMGh1Do02fQtESYCrlIT4f3Q4eeQ0GR3tpD4Wg1Z6kWQkDbAWGMvQX2f9WTAEbCZ0heY2e6eu5P
nMf0pm/q0qd9hP6TFPBwO7biSVTSPSGtBBpDtRijFI1g3kdWO+35xDyylv0+7EP3k+qPBdNmiTHW
hIYOSxuoLhNWW48K4mJ/rJ9pP8UtOqqH9/9BdNVwnc8WOEz7EZEk8so9zmdJXZstgV48CpFIAcO4
4NhxvV4vh8Gb42efdPlGaWk12Bzlm1CPk/4ZBQZadNaRnr/WgTiJLsulvVsxjnLH2BiXfsUpKG9U
PwP9foXX0OwkB+82i6w9+WKLB7E8OCWN6YeGE/U4JHO/SvzPx5PAy89UaLnG2J7Ld9s0hfbZo09K
iXxhOOj2/wCL5NQQcPKSSpCiCNe4tlICGMpu
解密: 既然如此， 了解清楚科学和人文谁更有意义到底是一种怎么样的存在，是解决一切问题的关键。 奥普拉·温弗瑞曾经提到过，你相信什么，你就成为什么样的人。这句话语虽然很短，但令我浮想联翩。 伏尔泰曾经说过，坚持意志伟大的事业需要始终不渝的精神。这不禁令我深思。 要想清楚，科学和人文谁更有意义，到底是一种怎么样的存在。 一般来讲，我们都必须务必慎重的考虑考虑。 我们不得不面对一个非常尴尬的事实，那就是， 这样看来。
```

## Demo 源码

[rsa_demo - mervynlam](https://github.com/mervynlam/demo/tree/master/rsa_demo)

## 参考资料

[Java 实现 RSA 非对称加密算法：生成密钥对、保存/读取密钥、加密/解密 - xietansheng](https://blog.csdn.net/xietansheng/article/details/88082266)

[RSA加密与解密(Java实现) - HFUT_qianyang](https://blog.csdn.net/qy20115549/article/details/83105736)

[RSA加密内容过长导致抛异常 - Mrdong916](https://blog.csdn.net/sinat_27938829/article/details/79935949)

