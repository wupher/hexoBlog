title:  iOS中的X509证书签名实现
date: 2013-8-16 14:46
tags: iOS, X509, PKCS, 密钥, 签名
---

# iOS平台下的X509密钥签名实现

## 基础概念
* X509: 一种证书格式，用于交换公钥、密钥等，里面存放了公钥、密钥信息。
* PKCS协议栈：RSA制定的一组公钥加密标准。经常用到的有PKCS#5，PKCS#7， PKCS#12，具体内容[请戳这里](http://baike.baidu.com/view/1477355.htm)。
* 公钥加密：用户散发公钥，自己持有密钥，需要传送加密信息的时候，使用公钥加密明文，只有持有密钥的人才能用密钥进行解密，阅读明文信息。
* 密钥签名：用户使用密钥对明文进行签名，所以持有公钥的人都可以使用公钥来验证这个签名，假如这个签名正确，就证明明文内容确实为密钥持有人发布的，没有经过第三方的篡改。

### 常见证书的种类
* pfx证书：PKCS#12用于公密钥交换的文件格式之一，Microsoft最早提出的专有格式，文件中同时含有公钥和密钥，有密码保护。
* p12证书：PKCS#12用于公密钥交换的文件格式之一，文件中同时含有公钥和密钥，有密码保护。
* cer证书：仅含有公钥，不含密钥，公钥以BASE64进行编码存放。
* pem证书：同cer证书一样，仅含有公钥，公钥以ASCII进行存放。

## 使用Security.framework来使用p12证书
Security.framework基本上全是C API格式的函数调用。操纵的数据格式也是C struct和 Core Foundation-style objects。像NSArray，NSDiction等都要通过*bridge*转换成CFArrayRef和CFDictionaryRef格式来使用。

### 读取p12证书
>
	OSStatus status = errSecSuccess;
	// 导入p12
	CFDataRef p12DataRef = ({
            NSData * p12Data = [NSData dataWithContentsOfFile:p12FilePath];
            (__bridge CFDataRef)p12Data;
        });
    NSMutableDictionary * options = [NSMutableDictionary dictionary];
    NSString * key = (__bridge NSString *)kSecImportExportPassphrase;
    options[key] = _password; 
    CFArrayRef items = CFArrayCreate(NULL, 0, 0, NULL);
    status = SecPKCS12Import(p12DataRef, (__bridge CFDictionaryRef)options, &items);
    if (status != errSecSuccess) {
        NSLog(@"P12证书导入失败，错误号：%ld", status);
        return;
    }
    
_password是一个NSString，存放的是p12证书的导入密码。 

#### 读取证书中的密钥
读取密钥需要先获取证书中的identity对象，然后再从identity中取出密钥：
> 
	CFDictionaryRef identityDict = CFArrayGetValueAtIndex(items, 0);
    SecIdentityRef identityRef = (SecIdentityRef)CFDictionaryGetValue(identityDict, kSecImportItemIdentity);    
    status = SecIdentityCopyPrivateKey(identityRef, &privateKeyRef);
    if (status != errSecSuccess) {
        NSLog(@"Private Key读取失败，错误号：%ld", status);
        return;
    }
    
#### 读取证书中的公钥
> 
	SecTrustRef trust = (SecTrustRef)CFDictionaryGetValue(identityDict, kSecImportItemTrust);
    publicKeyRef = SecTrustCopyPublicKey(trust);
    if (NULL == publicKeyRef) {
        NSLog(@"公钥获取失败");
        return;
    }
    
#### 使用密钥来对数据进行签名
考虑数据长度，并为了防止窃听方猜出明文来进行攻击，这里并不对明文直接进行签名，而是对明文的SHA1指纹进行签名。签名的padding相应使用的是kSecPaddingPKCS1SHA1。
>
	NSData * sha1 = [data getHashBytes];
    uint8_t * digestBytes = (uint8_t *)[sha1 bytes]; 
    uint8_t * signedBytes;
    size_t signedBytesSize = SecKeyGetBlockSize(privateKeyRef);
    signedBytes = malloc(signedBytesSize * sizeof(uint8_t));
    memset((void *)signedBytes, 0x0, signedBytesSize);
    OSStatus status = SecKeyRawSign(privateKeyRef, kSecPaddingPKCS1SHA1, digestBytes, CC_SHA1_DIGEST_LENGTH, signedBytes, &signedBytesSize);
    if (errSecSuccess != status) {
        NSLog(@"签名出现异常，错误号：%ld", status);
        return nil;
    }
    NSData * result = [NSData dataWithBytes:(const void *)signedBytes length:(NSUInteger)signedBytesSize];
    if (signedBytes) {
        free(signedBytes);
    }
    return result;

注意在使用SecKeyRawSign函数的时候，假如padding使用的是kSecPaddigPKCS1SHA1，则第四个参数，即被签名的数据长度必须设置为常用CC_SHA1_DIGEST_LENGTH，否则函数会报错，显示输入参数错误。

#### 代码下载
[示例代码](http://pan.baidu.com/share/link?shareid=780655914&uk=4161160899)


### Python的等价实现
objective-c折腾X509真是累的要死，同样的活，使用python语言来编写，只需要寥寥数行：
>
	from OpenSSL.crypto import *
	import base64
	p12 = load_pkcs12(file("./SentMobileClient.p12", 'rb').read(), "Sent,22856610.cn!2013")
	privateKey = p12.get_privatekey()
	data = "this is a test data"
	cipher = sign(privateKey, data, 'sha1')
	base = base64.standard_b64encode(cipher)
	print(base)
	
注意代码的最后一行，是将形成的签名做了base64编码。

### 另一个实现
最近因为项目的原因需要和支付宝做接口，意外发现支付宝的iOS接口框架里面封装了RSA签名功能。只要你获得公钥密钥（以NSString形式），即可使用对数据进行签名和RSAe加密。支付宝的实现是通过openSSL开源库来实现的，没有依赖于Apple的Security.framework。    
[支付宝SDK及DEMO下载](下载地址：http://download.alipay.com/public/api/base/WS_SECURE_PAY.zip)


