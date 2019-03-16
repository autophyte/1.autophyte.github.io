## Python3 RSA+AES混合加密（二）————加检验

[前文(Python实现rsa_aes混合加密)](https://autophyte.github.io/2019/03/15/PYTHON%E5%AE%9E%E7%8E%B0RSA_AES%E6%B7%B7%E5%90%88%E5%8A%A0%E5%AF%86.html)中已经实现了RSA+AES混合加密，但是这个加密是单向且没有进行安全校验的，也意味着，任何人都可以伪装成发送者向接收着发送伪装的信息，因为公钥是公开的。改进的方式就是发送方使用自已的么钥对数据进行签名

我们假定服务方名称为server，用户方为client

### 一般逻辑（与[前文](https://autophyte.github.io/2019/03/15/PYTHON%E5%AE%9E%E7%8E%B0RSA_AES%E6%B7%B7%E5%90%88%E5%8A%A0%E5%AF%86.html)稍有不同，注意粉红色的地方）

#### 加密过程（以server为例）

1. 准备好client的RSA公钥（注意，server使用client公钥：client_key_rsa_pub），用于对AES的密钥加密
2. <font color="Hotpink">准备好server的RSA私钥（server_key_rsa_pri），用于对数据进行签名</font>
3. <font color="Hotpink">使用server_key_rsa_pri对原始数据进行签名得到sign</font>
4. 随机生成AES加密密钥：key_aes_rand（这个会经过RSA加密发送给接收方）
5. 使用key_aes_rand对需要发送的数据msg进行AES加密（加密时需要对数据进行填充），得到encrypted_msg
6. 第3步aes加密后的数据有些是不可见数据，网络传输时可能会出错，此时会再进一步进行base64编码，得到base64ed_encrypted_msg
7. 对key_aes_rand使用key_rsa_pub进行RSA加密、base64，得到base64ed_encrypted_key_aes_rand
8. 把<font color="Hotpink">sign</font>、base64ed_encrypted_msg和base64ed_encrypted_key_aes_rand发送给接收方即可


#### 解密过程
1. 准备好client端自已RSA私钥(client_key_rsa_pri)，用于对AES的密钥进行解密
2. <font color="Hotpink">准备好server的RSA公钥（server_key_rsa_pub），用于对进行签名校验</font>
3. 对接收到的数据拆分，得到<font color="Hotpink">sign</font>、base64ed_encrypted_msg和base64ed_encrypted_key_aes_rand
4. 对base64ed_encrypted_key_aes_rand进行base64解密，得到encrypted_key_aes_rand
5. 对encrypted_key_aes_rand，使用RSA私钥key_rsa_pri解密，得到key_aes_rand
6. 对base64ed_encrypted_msg进行base64解密，得到encrypted_msg
7. 对encrypted_msg，使用key_aes_rand进行AES解密得到msg
8. <font color="Hotpink">使用server_key_rsa_pub对encrypted_msg和sign进行签名校验，校验成功说明不是伪造的数据</font>


### 上代码
```python
import base64
import os

# Crypto安装命令：`pip3 install -i https://pypi.douban.com/simple pycryptodome`
from Crypto import Random
from Crypto.Hash import SHA256
from Crypto.PublicKey import RSA
from Crypto.Signature import PKCS1_v1_5
from Crypto.Cipher import PKCS1_v1_5 as Cipher_pkcs1_v1_5
from Crypto.Cipher import AES


# AES根据16位对齐(即AES128)
BLOCK_SIZE = 16


# 伪随机数生成器
random_generator = Random.new().read


class RSAAES:
    def __init__(self):
        # 这里注意，发送方需要配置：发送方密钥，接收方公钥
        # 接收方需要配置：接收方密钥，发送方公钥
        self.key_private = b""  # 此为发送方私钥
        self.key_public = b""  # 此为发送方公钥

    @staticmethod
    def pad(s: bytes):
        return s + (BLOCK_SIZE - len(s) % BLOCK_SIZE) * \
                chr(BLOCK_SIZE - len(s) % BLOCK_SIZE).encode()

    @staticmethod
    def unpad(s: bytes):
        return s[:-ord(s[len(s) - 1:])]

    @staticmethod
    def get_random_aes_key(key_size: int = 16):
        """
        生成随机密钥
        """
        return base64.b64encode(os.urandom(int(key_size/4*3)))

    @staticmethod
    def decode_base64(data):
        missing_padding = 4 - len(data) % 4
        if missing_padding:
            data += b'=' * 3
        return base64.urlsafe_b64decode(data)

    @staticmethod
    def encode_base64(data):
        data = base64.urlsafe_b64encode(data)
        for i in range(3):
            if data.endswith(b'='):
                data = data[:-1]
        return data

    def sign_rsa(self, message: bytes):
        """
        非对称签名（此函发送方调用）
        :param message: 原始消息
        :return:
        """
        signer = PKCS1_v1_5.new(RSA.importKey(self.key_private))
        signature = signer.sign(SHA256.new(message))
        return self.encode_base64(signature)

    def verify_rsa(self, message: bytes, signature: bytes):
        """
        非对称验签（此函数接收方调用）
        :param message: 原始消息
        :param signature: 签名信息
        :return:
        """
        verifier = PKCS1_v1_5.new(RSA.importKey(self.key_public))
        signature = signature.rstrip(b'')
        if verifier.verify(SHA256.new(message), self.decode_base64(signature)):
            return True
        return False

    def encrypt(self, msg: bytes):
        # 生成随机AES密钥
        key_aes_rand = self.get_random_aes_key(BLOCK_SIZE)

        # 对原始信息进行AES加密再base64（需要网络传输选择urlsafe_b64encode）
        encrypted_msg = AES.new(
            key_aes_rand, AES.MODE_ECB).encrypt(self.pad(msg))
        base64ed_encrypted_msg = base64.urlsafe_b64encode(encrypted_msg)

        # 对AES随机密钥进行RSA加密
        encrypted_key_aes_rand = Cipher_pkcs1_v1_5.new(
            RSA.importKey(self.key_public)).encrypt(key_aes_rand)

        base64ed_encrypted_key_aes_rand = base64.urlsafe_b64encode(
            encrypted_key_aes_rand)

        sign = self.sign_rsa(msg)
        return {
            "sign": sign.decode(),
            "msg": base64ed_encrypted_msg.decode(),
            "key": base64ed_encrypted_key_aes_rand.decode()
        }

    def decrypt(self, source: dict):
        # 对接收到的数据拆分
        sign, base64ed_encrypted_msg, base64ed_encrypted_key_aes_rand = \
            source["sign"].encode(), source["msg"].encode(),\
            source["key"].encode()

        # 对base64ed_encrypted_key_aes_rand进行base64解密
        encrypted_key_aes_rand = base64.urlsafe_b64decode(
            base64ed_encrypted_key_aes_rand)

        # 对encrypted_key_aes_rand，使用RSA私钥key_rsa_pri解密，得到key_aes_rand
        key_aes_rand = Cipher_pkcs1_v1_5.new(
            RSA.importKey(self.key_private)).decrypt(
            encrypted_key_aes_rand, random_generator)

        encrypted_msg = base64.urlsafe_b64decode(base64ed_encrypted_msg)
        msg = self.unpad(AES.new(key_aes_rand, AES.MODE_ECB).decrypt(
            encrypted_msg))

        assert self.verify_rsa(msg, sign)
        return msg


class Sender(RSAAES):
    def __init__(self):
        super().__init__()
        self.key_public = b"""-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDW1aA3SG/czhFO+57bFVIMx390
TMWXUY1ZTHW6gXTaK1gBfd7rHVaxLhmOsfIBaxBFfXg6lOfBQ+9mOZDY0LPzRRP2
1pmW+DW/QvrAO1xL1t04zKzPNkslCpjQ3fSwwIyJnfQsRQxUxUv9j/OJdWh97XNc
K4EyACqRC9eO9vJ18QIDAQAB
-----END PUBLIC KEY-----"""
        self.key_private = b"""-----BEGIN PUBLIC KEY-----
MIICXQIBAAKBgQCsgzVEoL6Mwj92VH/ERMmyjC4dmG31WDAedALib1Z3mVCKOk5n
azTWJESgDp9UFV18LBveeEA1JRg5lk1wXRNShQ4A/q6FzIZ+vTB7FbXJC4vXzeq5
wor8Zj1j6JqQvz8sDnfvvdQqGfm3Rque51LWxeEcd+vRRDgLYqvsbwapDQIDAQAB
AoGAF5aFNRIJm/N/e+2H3s1NCuXR9GgAOPjK848HSfDRUN8cvRnF2Kw2+ETTQVNe
g7+8HZtmYB/vH5Un38/mXMPNPVRJK925kLC2/Qv2hzIX9x3KLzMeYpJ8FyE3MmWi
bHiKVOOB/cZ0LMQtjFgRYx8R162yJ9JiZ6/ddlMIlaf8kWECQQDwV6xC2VtAM8ZG
2mnL6RkkCywZ4ubsatO6SGiAq+4Y7uEtHgoc6f/rs9ftrQhcm+1lcY4julWmv5wn
ysY1SWw5AkEAt8BMBfvHUpiMU2F9g/0MEoveqQssxeFDOz70z8EYNYDYo/Yg7GwH
mmUN1tESzPcclYyCZocSP2tm5wjIRt/LdQJBAONeHZG0LHZFRKsczv9fyi/l/deT
Z2B7A0f0XiB0BjBCNHXJOEn4OOqTXY/0pLdvr5rLXWuBSKwSErk2RGJ+zkkCQCuv
hyOBCZFkfTAxpGKl3aHnKQethXaCKLbEL/XYpYXK3TaWBJvQzznwvoqM6Fhcg6o2
XqY7hKYZRby1xM+80yUCQQCFFLfjxTpVFXtExUKYHDMiREVOcOgfdiigBCztcC4W
W/YB6P7Tm9rGIapw2QqBRaHnAaeHmACKvdtnb1csdNA4
-----END PUBLIC KEY-----"""


class Receiver(RSAAES):
    def __init__(self):
        super().__init__()
        self.key_private = b"""-----BEGIN RSA PRIVATE KEY-----
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
        self.key_public = b"""-----BEGIN RSA PRIVATE KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCsgzVEoL6Mwj92VH/ERMmyjC4d
mG31WDAedALib1Z3mVCKOk5nazTWJESgDp9UFV18LBveeEA1JRg5lk1wXRNShQ4A
/q6FzIZ+vTB7FbXJC4vXzeq5wor8Zj1j6JqQvz8sDnfvvdQqGfm3Rque51LWxeEc
d+vRRDgLYqvsbwapDQIDAQAB
-----END RSA PRIVATE KEY-----"""


if __name__ == "__main__":
    import json

    sender, receiver = Sender(), Receiver()

    message = "这是一个server主动发送给client的测试数据"
    print("原始数据： {origin_msg}".format(origin_msg=message))
    value = sender.encrypt(message.encode("utf-8"))
    print("密文数据： {encrypt_msg}".format(
        encrypt_msg=json.dumps(value, indent=4)))
    msg = receiver.decrypt(value)
    print("解密数据： {decrypt_msg}".format(decrypt_msg=msg.decode("utf-8")))

    print("\n\n")
    message = "这是一个client返回给server的信息"
    print("原始数据： {origin_msg}".format(origin_msg=message))
    value = receiver.encrypt(message.encode("utf-8"))
    print("密文数据： {encrypt_msg}".format(
        encrypt_msg=json.dumps(value, indent=4)))
    msg = sender.decrypt(value)
    print("解密数据： {decrypt_msg}".format(decrypt_msg=msg.decode("utf-8")))

```

### 运行上面代码
![RSA/AES混合加密](/img/Python/python_rsa_aes_result2.png)

