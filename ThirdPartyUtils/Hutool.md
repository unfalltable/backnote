# 加密

## DES加密

- 直接用会有问题，需要先用Base64再加密，解密先用Base64再解密

```java
String salt = "";
SymmetricCrypto des = new SymmetricCrypto(SymmetricAlgorithm.DES, salt.getBytes());
//加密
String s = des.encryptHex("12345678");
//解密
String s1 = des.decryptStr(s);
```

## 枚举

- ContentType：常用Content-Type类型枚举

## 缓存

- LFUCache：最少使用率缓存
