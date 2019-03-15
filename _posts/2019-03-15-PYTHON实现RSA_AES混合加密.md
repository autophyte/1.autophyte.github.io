## PYTHON RSA+AES混合加密

为啥要用RSA+AES混合加密？安全和速度的妥协呗，RSA更安全但是速度实在太慢

### 一般逻辑顺序

#### 加密过程
1. 准备好RSA公钥（假设为key_rsa_pub）
2. 随机生成AES加密密钥：key_aes_rand（这个会经过RSA加密发送给接收方）
3. 使用key_aes_rand对需要发送的数据msg进行AES加密（加密时需要对数据进行填充），得到encrypted_msg
4. 第3步aes加密后的数据有些是不可见数据，网络传输时可能会出错，此时会再进一步进行base64编码，得到base64ed_encrypted_msg
5. 对key_aes_rand使用key_rsa_pub进行RSA加密、base64，得到base64ed_encrypted_key_aes_rand
6. 把base64ed_encrypted_msg和base64ed_encrypted_key_aes_rand发送给接收方即可

#### 解密过程
1. 准备好RSA私钥(key_rsa_pri)
2. 对接收到的数据拆分，得到base64ed_encrypted_msg和base64ed_encrypted_key_aes_rand
3. 对base64ed_encrypted_key_aes_rand进行base64解密，得到encrypted_key_aes_rand
4. 对encrypted_key_aes_rand，使用RSA私钥key_rsa_pri解密，得到key_aes_rand
5. 对base64ed_encrypted_msg进行base64解密，得到encrypted_msg
6. 对encrypted_msg，使用key_aes_rand进行AES解密得到msg

#### 说明
1. 以上流程实际上就是用随机生成的AES密钥对原始信息加密，然后用RSA公钥对AES的随机密钥加密，将两个加密后的数据发给接收端
2. 相比较直接AES加密的方式，混合加密时AES的密钥时随机生成的。（通过RSA加密后再发送给接收端）
3. RSA加密使用的公钥时公开的


### 上代码
```
import base64
import os

from Crypto import Random
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_v1_5 as Cipher_pkcs1_v1_5
from Crypto.Cipher import AES


# AES根据16位对齐(即AES128)
BLOCK_SIZE = 16


# 伪随机数生成器
random_generator = Random.new().read


# RSA公钥，发送方使用
public_key = b"""-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDW1aA3SG/czhFO+57bFVIMx390
TMWXUY1ZTHW6gXTaK1gBfd7rHVaxLhmOsfIBaxBFfXg6lOfBQ+9mOZDY0LPzRRP2
1pmW+DW/QvrAO1xL1t04zKzPNkslCpjQ3fSwwIyJnfQsRQxUxUv9j/OJdWh97XNc
K4EyACqRC9eO9vJ18QIDAQAB
-----END PUBLIC KEY-----"""


# RSA私钥，请保护好
private_key = b"""-----BEGIN RSA PRIVATE KEY-----
MIICWwIBAAKBgQDW1aA3SG/czhFO+57bFVIMx390TMWXUY1ZTHW6gXTaK1gBfd7r
HVaxLhmOsfIBaxBFfXg6lOfBQ+9mOZDY0LPzRRP21pmW+DW/QvrAO1xL1t04zKzP
NkslCpjQ3fSwwIyJnfQsRQxUxUv9j/OJdWh97XNcK4EyACqRC9eO9vJ18QIDAQAB
AoGARzKGZbaJ7AFsVw1TY4PjV1Izo9VgPnp2f7uz4IV6tmWwRX5I3F6IFoa9TZRy
LD970FZ5UTYmwDQa03lhzu5g/t4MBGZ+dEJ7hmcR4PJuLZSl59E2TzkEo9hE3DQ7
bq4wSIeBcY61hiwGf8tCnWetJbAvj+HS29HIB8sl1fu0jekCQQDgwiJm3HkgkoNq
mx5WliKvcGi+/eB9FXar9jn6762+mkge7htZbrM4tpIHl+5Vx+bn8E159FTcmgCS
My/WCb37AkEA9LJbn9GLvxupAcXq35snnq5xkNAsgHJHQaeu/VT0T2Gz5pUO3iB4
GLpWlia/E9eIDWj399Zs089AT72dAab0AwJAPtTmqxy9W+a5iEbe/1OvVJ43GhV8
+VrTtxT5dnYkeyFEQilMSf8RaSxYvHizrxVYLsTV098DDjybJkPa/pnwmwJAXeJk
5y/l92AseyKt2DdWfzqdFhvZRzsRfe5RZJ+I0UBCXxEH0FAS5CHygM/C9mD2sXZ5
1ZxuyuG04iN1LyIYcwJAULISlX5ZR8y+V6B1E/vndPu4BdfX5QORqzIwTAWsAt8k
uMsm5ubz+EEf4klTSVLhRZrxquVnTaBgiib6ngc9iA==
-----END RSA PRIVATE KEY-----"""


# 填充函数，填充模式：PKCS5Padding(填充内容等于缺少的字节数)
pad = lambda s: s + (BLOCK_SIZE - len(s) % BLOCK_SIZE) * \
                chr(BLOCK_SIZE - len(s) % BLOCK_SIZE).encode()
# 返填充函数
unpad = lambda s: s[:-ord(s[len(s) - 1:])]


# 生成随机密钥
def get_random_key_readable(key_size=16):
    ulen = int(key_size/4*3)
    key = base64.b64encode(os.urandom(ulen))
    return key


def encrypt(msg: bytes, key_rsa_pub: bytes):
    # 生成随机AES密钥
    key_aes_rand = get_random_key_readable(BLOCK_SIZE)

    # 对原始信息进行AES加密再base64（需要网络传输选择urlsafe_b64encode）
    encrypted_msg = AES.new(key_aes_rand, AES.MODE_ECB).encrypt(pad(msg))
    base64ed_encrypted_msg = base64.urlsafe_b64encode(encrypted_msg)

    # 对AES随机密钥进行RSA加密
    encrypted_key_aes_rand = Cipher_pkcs1_v1_5.new(
        RSA.importKey(key_rsa_pub)).encrypt(key_aes_rand)

    base64ed_encrypted_key_aes_rand = base64.urlsafe_b64encode(encrypted_key_aes_rand)
    return [base64ed_encrypted_msg, base64ed_encrypted_key_aes_rand]


def decrypt(source: list, key_rsa_pri: bytes):
    # 对接收到的数据拆分，得到base64ed_encrypted_msg和base64ed_encrypted_key_aes_rand
    base64ed_encrypted_msg, base64ed_encrypted_key_aes_rand = source[0], source[1]

    # 对base64ed_encrypted_key_aes_rand进行base64解密，得到encrypted_key_aes_rand
    encrypted_key_aes_rand = base64.urlsafe_b64decode(base64ed_encrypted_key_aes_rand)

    # 对encrypted_key_aes_rand，使用RSA私钥key_rsa_pri解密，得到key_aes_rand
    key_aes_rand = Cipher_pkcs1_v1_5.new(RSA.importKey(key_rsa_pri)).decrypt(
        encrypted_key_aes_rand, random_generator)

    encrypted_msg = base64.urlsafe_b64decode(base64ed_encrypted_msg)
    msg = unpad(AES.new(key_aes_rand, AES.MODE_ECB).decrypt(encrypted_msg))
    return msg


if __name__ == "__main__":
    message = "这是一个测试数据"
    value = encrypt(message.encode("utf-8"), key_rsa_pub=public_key)
    msg = decrypt(value, key_rsa_pri=private_key).decode()

    print("原始数据： {origin_msg}".format(origin_msg=message))
    print("密文数据： {encrypt_msg}".format(encrypt_msg=value))
    print("解密数据： {decrypt_msg}".format(decrypt_msg=msg))
```

