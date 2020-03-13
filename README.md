# Encrypt-Signature

## ascii编码：
> 字符转ascii : Char -> toInt -> Int 

> 凯撒加密算法：（移动字符）
* 1、字符转ascii码；
* 2、ascii码 + key（key是加密的数字，如：1）
* 3、ascii码转成字符；

> 凯撒加密算法破解：
> 频率分析法：
* “e”字符出现的频率最高，一般假以“e”为参考
* 1、在加密的密文中找处出现最多的几个字符；
* 2、逐个字符的ascii码 与 “e”的ascii码比较，并尝试解密；

> 对称加密算法：（操作二进制）
> AES比DES高级一点

> DES：
> （加密）
* 1、
```
val cipher = Cipher.getInstance("DES");
```
* 2、
```
val kf = SecretKeyFactory.getInstance("DES");
val keySpec = DESKeySpec(new Byte[]);   (Byte[] 是加密钥匙，需要8位字符)
val key = kf.generateSecret(keySpec);
cipher.init(Cipher.ENCRYPT_MODE,key); 
```
* 3、
```
cipher.doFinal(input);  (input  需要加密字符串的字符数组)
```

> （解密）
* 1、
```
val cipher = Cipher.getInstance("DES");
```
* 2、
```
val kf = SecretKeyFactory.getInstance("DES");
val keySpec = DESKeySpec(new Byte[]);   (Byte[] 是加密钥匙，需要8位字符)
val key = kf.generateSecret(keySpec);
cipher.init(Cipher.DECRYPT_MODE,key); 
```
* 3、
```
cipher.doFinal(input);  (input  需要解密字符串的字符数组)
```

> AES：
> （加密）
* 1、
```
val cipher = Cipher.getInstance("AES");
```
* 2、
```
val keySpec = SecretKeySpec(new Byte[],"AES");   (Byte[] 是加密钥匙，需要16位字符)
cipher.init(Cipher.ENCRYPT_MODE, keySpec); 
```
* 3、
```
cipher.doFinal(input);  (input  需要加密字符串的字符数组)
```

> （解密）
* 1、
```
val cipher = Cipher.getInstance("AES");
```
* 2、
```
val keySpec = SecretKeySpec(new Byte[],"AES");   (Byte[] 是加密钥匙，需要8位字符)
cipher.init(Cipher.DECRYPT_MODE, keySpec );
```
* 3、
```
cipher.doFinal(input);  (input  需要解密字符串的字符数组)
```

* 加密/解密为了避免中文乱码，需要使用base64编码和解码
* DES和AES区别：DES需要8位密钥，而AES需要16位；

> 工作模式：
> ECB （电子密码本）：并行加密
> CBC（密码分组链接）：串行加密  ，需要在init方法添加额外参数：cipher.init(Cipher.ENCRYPT_MODE, keySpec，IvParameterSpec(new Byte[])); 


> 非对称加密
> RSA：公钥加密，私钥解密；私钥加密，公钥解密

* 生成密钥对：
```
val generator = KeyPairGenerator.getInstance("RSA")
val keyPair = generator.getKeyPair()
val publicKey = keyPair .public
val privateKey = keyPair .private
```

* 加密的数据的长度不能大于117个字节，解决方式：采用分段加密；

* 解密的数据的长度不能大于128个字节，解决方式：采用分段解密；

* 使用保存好的密钥：
```
val publicKeyStr = "xxxxx";
val privateKeyStr = "xxx";

val kf = KeyFactory.getInstance("RSA")
val privateKey = kf.generatePrivate(PKCS8EncodeKeySpec(privateKeyStr))
val publicKey = kf.generatePublic(X509EncodeKeySpec(publicKeyStr))
```

> 消息摘要：
> MD5：
```
val digest = MessageDigest.getInstance("MD5")
val result = digest.digest(input)    //input是字符数组；加密后result是16个字节

val stringBuilder = StringBuilder()

//转16进制
result.forEach{
	val value = it
	val hex = value.toInt() and (0xFF)  // and：表示位与运算
	val hexStr = Integer.toHexString(hex)
	if(hexStr.length == 1){
		stringBuilder.append("0").append(hexStr)
	}else{
		stringBuilder.append(hexStr)
	}
}

stringBuilder .toString() 
//加密后 16个字节
//转16进制后  32个字节
```

> SHA1:
```
val digest = MessageDigest.getInstance("SHA-1")
//加密后 20个字节
//转16进制后 40个字节

SHA256:
val digest = MessageDigest.getInstance("SHA-256")
//加密后 32个字节
//转16进制后 64个字节
```

> 数字签名：
```
val signature = Signature.getInstance("SHA256withRSA")
signature.initSign(privateKey)  //  privateKey / publicKey  密钥/公钥
signature.update(new Byte[])  // Byte[] 需要签名的字符数组
val sign = signature.sign()
sign = Base64.encode(sign)
```

> 校验：
```
val signature = Signature.getInstance("SHA256withRSA")
signature.initVerify(publicKey)  //  privateKey / publicKey  密钥/公钥
signature.update(new Byte[])  // Byte[] 需要签名的字符数组
val verify = signature.verify(Base64.decode(sign))
```

> 数字签名流程：
* 1、明文 -> 数字签名（SHA256）-> 密钥加密 -> 签名值A；
* 2、明文 + 签名值A 发送到服务端；
* 3、使用公钥解密 签名A ->得到 签值名B；
* 4、明文 -> 数字签名（SHA256）-> 密钥加密 -> 签名值C；
* 5、签值名B == 签名值C 证明数据没有被修改；
