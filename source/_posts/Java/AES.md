```java
import java.util.Arrays;
import java.lang.Throwable;

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;

import org.apache.axis.encoding.Base64;

public class aesEncryption {
    private static final String CYPHER_MODE = "AES/CBC/NoPadding";

    public static byte[] encrypt(byte[] key, byte[] initVector, byte[] value) {
        try {
            IvParameterSpec iv = new IvParameterSpec(initVector);
            SecretKeySpec skeySpec = new SecretKeySpec(key, "AES");

            Cipher cipher = Cipher.getInstance(CYPHER_MODE);
            cipher.init(Cipher.ENCRYPT_MODE, skeySpec, iv);
            int blockSize = cipher.getBlockSize();
            byte[] plaintext = padding(value, blockSize);
            return cipher.doFinal(plaintext);
        } catch (Exception ex) {
            ex.printStackTrace();
        }

        return null;
    }

    public static byte[] decrypt(byte[] key, byte[] initVector, byte[] encrypted) {
        try {
            IvParameterSpec iv = new IvParameterSpec(initVector);
            SecretKeySpec skeySpec = new SecretKeySpec(key, "AES");

            Cipher cipher = Cipher.getInstance(CYPHER_MODE);
            cipher.init(Cipher.DECRYPT_MODE, skeySpec, iv);

            return unpadding(cipher.doFinal(encrypted));
        } catch (Exception ex) {
            ex.printStackTrace();
        }

        return null;
    }

    private static byte[] padding(byte[] value, int blockSize) {
        int plaintextLength = value.length;
        if (plaintextLength % blockSize != 0) {
            plaintextLength = plaintextLength + (blockSize - (plaintextLength % blockSize));
        }
        byte[] plaintext = new byte[plaintextLength];
        System.arraycopy(value, 0, plaintext, 0, value.length);
        return plaintext;
    }

    private static byte[] unpadding(byte[] bytes) {
        int i = bytes.length - 1;
        while (i >= 0 && bytes[i] == 0)
        {
            --i;
        }

        return Arrays.copyOf(bytes, i + 1);
    }

    public static void main(String[] args) {
        try {
            byte[] key = "keyskeyskeyskeys".getBytes();
            byte[] iv = "keyskeyskeyskeys".getBytes();
            byte[] content = "123".getBytes("utf-8");
            byte[] cyphertext = encrypt(key, iv, content);
            String b64 = Base64.encode(cyphertext);
            System.out.println(b64);
            byte[] de_b64 = decrypt(key, iv, Base64.decode(b64));
            String plaintext = new String(de_b64);
            System.out.println(plaintext);
        } catch (Throwable t) {
            t.printStackTrace();
        }
    }
}
```

```py
from Crypto.Cipher import AES
import base64


class AESEncrypt:
    def __init__(self, key, iv):
        self.key = key
        self.iv = iv
        self.mode = AES.MODE_CBC

    def encrypt(self, text):
        cryptor = AES.new(self.key, self.mode, self.key)
        length = AES.block_size
        text_pad = self.padding(length, text)
        ciphertext = cryptor.encrypt(text_pad)
        cryptedStr = str(base64.b64encode(ciphertext), encoding='utf-8')
        return cryptedStr

    def padding(self, length, text):
        count = len(text.encode('utf-8'))
        if count % length != 0:
            add = length - (count % length)
        else:
            add = 0
        text1 = text + ('\0' * add)
        return text1

    def decrypt(self, text):
        base_text = base64.b64decode(text)
        cryptor = AES.new(self.key, self.mode, self.key)
        plain_text = cryptor.decrypt(base_text)
        ne = plain_text.decode('utf-8').rstrip('\0')
        return ne


if __name__ == '__main__':
    aes_encrypt = AESEncrypt(key='keyskeyskeyskeys', iv="keyskeyskeyskeys")  # 初始化key和IV
    text = '123'
    sign_data = aes_encrypt.encrypt(text)
    print(sign_data)
    data = aes_encrypt.decrypt(sign_data)
    print(data)
```