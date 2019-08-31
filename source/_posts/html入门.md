---
title: RSA2 AES
date: 2019-02-13 14:41:48
tags:
---



```
    import android.util.Base64;
    import android.util.Log;

    import java.io.File;
    import java.io.FileInputStream;
    import java.io.FileOutputStream;
    import java.nio.ByteBuffer;
    import java.nio.channels.FileChannel;
    import java.security.KeyFactory;
    import java.security.PrivateKey;
    import java.security.PublicKey;
    import java.security.Signature;
    import java.security.spec.PKCS8EncodedKeySpec;
    import java.security.spec.X509EncodedKeySpec;
    import java.util.Arrays;

    import javax.crypto.Cipher;
    import javax.crypto.spec.IvParameterSpec;
    import javax.crypto.spec.SecretKeySpec;


    public class AESUtils {

        private static final String TAG = "AESUtils";


        public static void decryptOTAFile(File otaFile, File publicKey, File privateKey, String outFile) {

            ByteBuffer buffer = loadFile(otaFile);
            if (buffer == null) {
                Log.e(TAG, "otaFile read fail");
                return;
            }

            buffer.rewind();
            byte[] sign = new byte[256];
            buffer.get(sign);
            buffer.mark();

            byte[] keyAndData = new byte[buffer.remaining()];
            buffer.get(keyAndData);

            buffer.reset();
            byte[] key = new byte[256];
            buffer.get(key);

            byte[] data = new byte[buffer.remaining()];
            buffer.get(data);

            // 1、验证签名
            if (!verifySign(publicKey, sign, keyAndData)) {
                return;
            }

            // 2、RSA解密key
            byte[] keyAndIv = rsaDecrypt(privateKey, key);
            if (keyAndIv == null) {
                return;
            }
            byte[] k = new byte[16];
            byte[] iv = new byte[16];
            System.arraycopy(keyAndIv, 0, k, 0, 16);
            System.arraycopy(keyAndIv, 16, iv, 0, 16);
            Arrays.fill(keyAndIv, (byte) 0);

            Log.i(TAG, "RSA解码KEY  成功");

            // 3、AES解码文件
            byte[] bytes = aesDecrypt(k, iv, data);

            Arrays.fill(k, (byte) 0);
            Arrays.fill(iv, (byte) 0);

            Log.i(TAG, "AES解码数据  成功");

            writeFile(bytes, outFile);
        }


        private static ByteBuffer loadFile(File file) {
            ByteBuffer buffer = null;
            try (FileInputStream fin = new FileInputStream(file)) {
                FileChannel channel = fin.getChannel();
                byte[] array = new byte[fin.available()];
                buffer = ByteBuffer.wrap(array);
                channel.read(buffer);
                channel.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
            return buffer;
        }


        private static void writeFile(byte[] data, String outFile) {
            try (FileOutputStream fos = new FileOutputStream(outFile)) {
                FileChannel channel = fos.getChannel();
                channel.write(ByteBuffer.wrap(data));
                channel.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        /**
        * 验签
        *
        * @param key       公钥
        * @param signature 签名
        * @param data      数据
        */
        private static boolean verifySign(File key, byte[] signature, byte[] data) {
            boolean verifySignSuccess = false;
            try {

                ByteBuffer byteBuffer = loadFile(key);
                byte[] array = byteBuffer.array();
                PublicKey publicKey = loadPublicKey(array);
                Signature verifySign = Signature.getInstance("SHA256withRSA");
                verifySign.initVerify(publicKey);
                verifySign.update(data);
                verifySignSuccess = verifySign.verify(signature);

                Log.i(TAG, "SHA-256 验签结果：" + verifySignSuccess);

                // 擦除内存中的公钥数据
                Arrays.fill(array, (byte) 0);

            } catch (Exception e) {
                e.printStackTrace();
            }
            return verifySignSuccess;
        }


        /**
        * RSA 解码
        *
        * @param privateKeyFile 私钥 PKCS8格式
        * @param encryptedData  加密的数据
        */
        private static byte[] rsaDecrypt(File privateKeyFile, byte[] encryptedData) {
            try {
                ByteBuffer byteBuffer = loadFile(privateKeyFile);
                byte[] array = byteBuffer.array();
                PrivateKey privateKey = loadPrivateKey(array);

                // 擦除内存中的密钥数据
                Arrays.fill(array, (byte) 0);

                Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
                cipher.init(Cipher.DECRYPT_MODE, privateKey);
                return cipher.doFinal(encryptedData);
            } catch (Exception e) {
                e.printStackTrace();
            }
            return null;
        }


        /**
        * AES 解码
        */
        private static byte[] aesDecrypt(final byte[] key, final byte[] iv, final byte[] data) {
            byte[] bytes = null;
            try {
                Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
                cipher.init(Cipher.DECRYPT_MODE, new SecretKeySpec(key, "AES"), new IvParameterSpec(iv));
                bytes = cipher.doFinal(data);
            } catch (Exception e) {
                e.printStackTrace();
            }
            return bytes;
        }


        /**
        * 解码 PrivateKey
        */
        private static PrivateKey loadPrivateKey(byte[] privateKey) {
            try {
                KeyFactory keyFactory = KeyFactory.getInstance("RSA");

                String temp = new String(privateKey);
                String privateKeyPEM = temp.replace("-----BEGIN PRIVATE KEY-----\n", "");
                privateKeyPEM = privateKeyPEM.replace("-----END PRIVATE KEY-----", "");
                byte[] decode = Base64.decode(privateKeyPEM, Base64.DEFAULT);
                PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(decode);

                // 擦除内存中的数据
                Arrays.fill(decode, (byte) 0);

                return keyFactory.generatePrivate(keySpec);
            } catch (Exception e) {
                e.printStackTrace();
            }
            return null;
        }


        /**
        * 解码 PublicKey
        */
        private static PublicKey loadPublicKey(byte[] publicKey) {
            try {

                KeyFactory keyFactory = KeyFactory.getInstance("RSA");

                String temp = new String(publicKey);
                String publicKeyPEM = temp.replace("-----BEGIN PUBLIC KEY-----\n", "");
                publicKeyPEM = publicKeyPEM.replace("-----END PUBLIC KEY-----", "");

                byte[] decode = Base64.decode(publicKeyPEM, Base64.DEFAULT);
                X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(decode);

                // 擦除内存中的数据
                Arrays.fill(decode, (byte) 0);

                return keyFactory.generatePublic(x509EncodedKeySpec);
            } catch (Exception e) {
                e.printStackTrace();
            }
            return null;
        }

    }
```