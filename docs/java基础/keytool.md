### 非堆成加密

#### RSA | ECC | SDA | DH

[加解密篇 - 非对称加密算法 (RSA、DSA、ECC、DH)](https://blog.csdn.net/u014294681/article/details/86705999)

##### 注意

> java 中实现了RSA，ECC需要借助三方包 `bcprov-ext-jdk15on.jar` (本文使用版本 1.60) 



### 创建证书库（BKS）

```shell
keytool -genseckey -alias custody -keystore custodyKeyStore.bks -providerclass org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath C:\work\util\repository\org\bouncycastle\bcprov-jdk15on\1.60\bcprov-jdk15on-1.60.jar -storetype BKS
```



### 查看证书库（BKS）

```shell
keytool -list -v -keystore custodyKeyStore.bks -storepass custody -providerclass org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath C:\work\util\repository\org\bouncycastle\bcprov-jdk15on\1.60\bcprov-jdk15on-1.60.jar -storetype BKS
```

### 生成密钥对

```shell
keytool -genkeypair -alias custody -keyalg EC -keysize 256 -sigalg SHA512withECDSA -dname "CN=W,OU=W,O=W,L=CD,ST=SC,C=CN" -validity 365 -keystore custodyKeyStore.bks -storetype BKS -providerclass org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath C:\work\util\repository\org\bouncycastle\bcprov-jdk15on\1.60\bcprov-jdk15on-1.60.jar -v -keypass custody
```

### 生成  ECC  证书

```shell
keytool -keystore custody.jks -genkey -keyalg EC -keysize 256 -sigalg SHA512withECDSA -keypass custody -validity 365 -storepass custody
```

> Warning:
> JKS 密钥库使用专用格式。建议使用 "keytool -importkeystore -srckeystore custody.jks -destkeystore custody.jks -deststoretype pkcs12" 迁移到行业标准格式 PKCS12。zhi

##### 解决办法

```shell
keytool -importkeystore -srckeystore custody.jks -destkeystore custody.jks -deststoretype pkcs12
```



### 证书导入 `cert` 

```shell
keytool -keystore custody.jks -export -file custody.cert 
```



### 导入 `cert` 到证书库（BKS）

```shell
keytool -keystore custodyKeyStore.bks -import -file custody.cert -providerclass org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath C:\work\util\repository\org\bouncycastle\bcprov-jdk15on\1.60\bcprov-jdk15on-1.60.jar -storetype BKS
```



### 删除证书（BKS）

```shell
 keytool -delete -alias custody -keystore custodyKeyStore.bks -storepass custody -providerclass org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath C:\work\util\repository\org\bouncycastle\bcprov-jdk15on\1.60\bcprov-jdk15on-1.60.jar -storetype BKS
```



### Java

### 向 keystore 写入密钥对

```java
static private final Provider PROVIDER;

static {
    PROVIDER = new BouncyCastleProvider();
    Security.addProvider(PROVIDER);
}

public static load() {
	KeyStore keyStore = KeyStore.getInstance("BKS", PROVIDER);
        
    String pass = "pass";
    String path = "KeyStore.bks";
    char[] passBytes = pass.toCharArray();
    String pri = "MEECAQAwEwYHKoZIzj0CAQYIKoZIzj0DAQcEJzAlAgEBBCBKgfotOY9vDbrCqpZRLnq8sdo8/Vdj\n" +
        "FzfVJlMxq32FMA==";
	// 构建私钥
    PrivateKey privateKey = getPrivateKeyFromString(pri);

    try (FileInputStream fin = new FileInputStream(path)) {
        // 初始化私钥库
        keyStore.load(fin, passBytes);
		// 添加私钥
        keyStore.setKeyEntry("test", privateKey, passBytes, new X509Certificate[] { newCertificate("test") });
        try (FileOutputStream fos = new FileOutputStream(path)) {
            // 导入私钥到私钥库
            keyStore.store(fos, passBytes);
        }
    } catch (CertificateException e) {
        e.printStackTrace();
    }
}

private static PrivateKey getPrivateKeyFromString(String priKeyStr) throws NoSuchAlgorithmException, InvalidKeySpecException, IOException {
    KeyFactory keyFactory = KeyFactory.getInstance("EC");
    //将BASE64编码的私钥字符串进行解码
    BASE64Decoder decoder = new BASE64Decoder();
    byte[] encodeByte = decoder.decodeBuffer(priKeyStr);
    //将BASE64解码后的字节数组，构造成PKCS8EncodedKeySpec对象，生成私钥对象
    return keyFactory.generatePrivate(new PKCS8EncodedKeySpec(encodeByte));
}

private static X509Certificate newCertificate(String name) throws NoSuchProviderException, NoSuchAlgorithmException, InvalidKeyException, IOException, CertificateException, SignatureException {
    CertAndKeyGen certGen = new CertAndKeyGen("EC", "SHA256WithECDSA");
    certGen.generate(256);
    long validSecs = (long) 365 * 24 * 60 * 60;
    return certGen.getSelfCertificate(new X500Name("CN=" + name), validSecs);
}
```

### 签名和验签

```java
public static void main(String[] args) throws Exception {
    FileInputStream is = new FileInputStream(new File("C:\\work\\workspace\\oms\\code\\oms-web\\src\\main\\resources\\tomcat.keystore"));

    KeyStore keyStore = KeyStore.getInstance("JKS");
    //这里填设置的keystore密码，两个可以不一致
    keyStore.load(is, "tomcat".toCharArray());

    //加载别名，这里认为只有一个别名，可以这么取；当有多个别名时，别名要用参数传进来。不然，第二次的会覆盖第一次的
    String keyAlias = "mykey" ;

    Certificate certificate = keyStore.getCertificate(keyAlias);

    //加载公钥
    PublicKey publicKey = certificate.getPublicKey();
    //加载私钥,这里填私钥密码
    PrivateKey privateKey = ((KeyStore.PrivateKeyEntry) keyStore.getEntry(keyAlias,
                                                                          new KeyStore.PasswordProtection("tomcat".toCharArray()))).getPrivateKey();

    //base64输出私钥
    String strKey = Base64.encodeBase64String(privateKey.getEncoded());
    System.out.println(strKey);

    //测试签名
    String sign = Base64.encodeBase64String(sign("测试msg".getBytes(),privateKey,"SHA256withRSA",null));
    //测试验签
    boolean verfi = verify("测试msg".getBytes(),Base64.decodeBase64(sign), publicKey,"SHA256withRSA",null);
    System.out.println(verfi);
}

/**
     * 签名
     */
public static byte[] sign(byte[] message, PrivateKey privateKey, String algorithm, String provider) throws Exception {
    Signature signature;
    if (null == provider || provider.length() == 0) {
        signature = Signature.getInstance(algorithm);
    } else {
        signature = Signature.getInstance(algorithm, provider);
    }
    signature.initSign(privateKey);
    signature.update(message);
    return signature.sign();
}
/**
     * 验签
     */
public static boolean verify(byte[] message, byte[] signMessage, PublicKey publicKey, String algorithm,
                             String provider) throws Exception {
    Signature signature;
    if (null == provider || provider.length() == 0) {
        signature = Signature.getInstance(algorithm);
    } else {
        signature = Signature.getInstance(algorithm, provider);
    }
    signature.initVerify(publicKey);
    signature.update(message);
    return signature.verify(signMessage);
}
```

### 一些问题

#### KeyStore loading Problem when using the Bouncycastle Provider #306

```java
// keystore 是 PKCS12，key 是 ec 256 SHA256WITHECDSA 
try (InputStream inputStream = new FileInputStream(new File("src\\integrationTest\\test.p12"))) { 	     keyStore = KeyStore.getInstance("PKCS12","BC"); keyStore.load(inputStream, "test".toCharArray()); 
}
```

```
java.io.IOException: exception unwrapping private key - java.security.InvalidKeyException: pad block corrupted
```

异常原因

BC 实现的 PKCS12 要求 keystore 和 key 的密码一样

