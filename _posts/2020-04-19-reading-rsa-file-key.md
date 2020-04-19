---
layout: post
title:  "Securing webservices using certificates"
date:   2020-04-11 13:27:34 -0700
categories: secure spring security rest service using certificates
---
## How to Read a RSA File to private and public keys


Generate the RSA file using following commands
```
ssh-keygen -t rsa -b 2048 -m PEM -f ../src/main/resources/s2sauth.key
openssl rsa -in ../src/main/resources/s2sauth.key -pubout -outform PEM -out ../src/main/resources/s2sauth.key.pub
```

Use the following code to convert this to a Private and public key. To be used for Jwt token decryption/encryption

```
package test.util;

import org.bouncycastle.jce.provider.BouncyCastleProvider;
import org.bouncycastle.util.io.pem.PemObject;
import org.bouncycastle.util.io.pem.PemReader;

import java.io.*;
import java.security.KeyFactory;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.Security;
import java.security.spec.EncodedKeySpec;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;

public class PemUtils {

    private static byte[] parsePEMFromReader(Reader inputReader) throws IOException {
        Security.addProvider(new BouncyCastleProvider());
        PemReader reader = new PemReader(inputReader);
        PemObject pemObject = reader.readPemObject();
        byte[] content = pemObject.getContent();
        reader.close();
        return content;

    }

    private static byte[] parsePEMFile(File pemFile) throws IOException {
        if (!pemFile.isFile() || !pemFile.exists()) {
            throw new FileNotFoundException(String.format("The file '%s' doesn't exist.", pemFile.getAbsolutePath()));
        }
        return parsePEMFromReader(new FileReader(pemFile));
    }

    private static byte[] parsePEMString(String inputFileContent) throws IOException {
        if (inputFileContent == null || inputFileContent.length() == 0) {
            throw new RuntimeException("Empty or null input content");
        }
        return parsePEMFromReader(new StringReader(inputFileContent));
    }

    private static PublicKey getPublicKey(byte[] keyBytes, String algorithm) {
        PublicKey publicKey = null;
        try {
            KeyFactory kf = KeyFactory.getInstance(algorithm);
            EncodedKeySpec keySpec = new X509EncodedKeySpec(keyBytes);
            publicKey = kf.generatePublic(keySpec);
        } catch (Exception x) {
            throw new RuntimeException(x);
        }

        return publicKey;
    }

    private static PrivateKey getPrivateKey(byte[] keyBytes, String algorithm) {
        PrivateKey privateKey = null;
        try {
            KeyFactory kf = KeyFactory.getInstance(algorithm);
            EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(keyBytes);
            privateKey = kf.generatePrivate(keySpec);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

        return privateKey;
    }

    public static PublicKey readPublicKeyFromFile(String filepath, String algorithm) throws IOException {
        byte[] bytes = PemUtils.parsePEMFile(new File(filepath));
        return PemUtils.getPublicKey(bytes, algorithm);
    }

    public static PublicKey readPublicKeyFromContent(String content, String algorithm) throws IOException {
        byte[] bytes = PemUtils.parsePEMString(content);
        return PemUtils.getPublicKey(bytes, algorithm);
    }


    public static PrivateKey readPrivateKeyFromFile(String filepath, String algorithm) throws IOException {
        byte[] bytes = PemUtils.parsePEMFile(new File(filepath));
        return PemUtils.getPrivateKey(bytes, algorithm);
    }

    public static PrivateKey readPrivateKeyFromContent(String content, String algorithm) throws IOException {
        byte[] bytes = PemUtils.parsePEMString(content);
        return PemUtils.getPrivateKey(bytes, algorithm);
    }

}
```





#Reference

https://medium.com/@niral22/2-way-ssl-with-spring-boot-microservices-2c97c974e83

https://github.com/niral22/two-way-ssl-spring-boot