#### Shiro-1.2.4

##### 生成cookie

登陆成功调用onSuccessfulLogin，清除旧的身份信息，如果勾选了RememberMe则调用rememberIdentity，将调用rememberIdentity

```java
    public void onSuccessfulLogin(Subject subject, AuthenticationToken token, AuthenticationInfo info) {
        //always clear any previous identity:
        forgetIdentity(subject);

        //now save the new identity:
        if (isRememberMe(token)) {
            rememberIdentity(subject, token, info);
        } else {
            if (log.isDebugEnabled()) {
                log.debug("AuthenticationToken did not indicate RememberMe is requested.  " +
                        "RememberMe functionality will not be executed for corresponding account.");
            }
        }
    }
```

rememberIdentity：

```java
public void rememberIdentity(Subject subject, AuthenticationToken token, AuthenticationInfo authcInfo) {
    PrincipalCollection principals = getIdentityToRemember(subject, authcInfo);
    rememberIdentity(subject, principals);
}
```

getIdentityToRemember获取身份信息，并赋值到principals，然后调用了另一个rememberIdentity

```java
protected void rememberIdentity(Subject subject, PrincipalCollection accountPrincipals) {
    byte[] bytes = convertPrincipalsToBytes(accountPrincipals);
    rememberSerializedIdentity(subject, bytes);
}
```

convertPrincipalsToBytes将上一步得到的principals转为字节数组，看convertPrincipalsToBytes

```java
    protected byte[] convertPrincipalsToBytes(PrincipalCollection principals) {
        byte[] bytes = serialize(principals);
        if (getCipherService() != null) {
            bytes = encrypt(bytes);
        }
        return bytes;
    }
```

先将principals序列化，然后判断CipherService是否为空（在AbstractRememberMeManager的构造函数中，会new一个CipherService，一般不为空），若不为空则对序列化的内容加密，encrypt函数

```java
    protected byte[] encrypt(byte[] serialized) {
        byte[] value = serialized;
        CipherService cipherService = getCipherService();
        if (cipherService != null) {
            ByteSource byteSource = cipherService.encrypt(serialized, getEncryptionCipherKey());
            value = byteSource.getBytes();
        }
        return value;
    }
```

调用了cipherService的encrypt（cipherService有不同的实现方式，其中AesCipherService是一个种类），传入的第一个参数为序列化的内容，第二个参数

```
public byte[] getEncryptionCipherKey() {
    return encryptionCipherKey;
}
```

encryptionCipherKey是其成员变量，在构造函数中

```
public AbstractRememberMeManager() {
    this.serializer = new DefaultSerializer<PrincipalCollection>();
    this.cipherService = new AesCipherService();
    setCipherKey(DEFAULT_CIPHER_KEY_BYTES);
}
```

setCipherKey函数，将加密的密钥和解密的密钥全部设为DEFAULT_CIPHER_KEY_BYTES

```java
    public void setCipherKey(byte[] cipherKey) {
        //Since this method should only be used in symmetric ciphers
        //(where the enc and dec keys are the same), set it on both:
        setEncryptionCipherKey(cipherKey);
        setDecryptionCipherKey(cipherKey);
    }
```

setEncryptionCipherKey和setDecryptionCipherKey

```java
	public void setEncryptionCipherKey(byte[] encryptionCipherKey) {
		this.encryptionCipherKey = encryptionCipherKey;
	}
	public void setDecryptionCipherKey(byte[] decryptionCipherKey) {
		this.decryptionCipherKey = decryptionCipherKey;
	}
```

DEFAULT_CIPHER_KEY_BYTES为静态变量

```
private static final byte[] DEFAULT_CIPHER_KEY_BYTES = Base64.decode("kPH+bIxk5D2deZiIxcaaaA==");
```

cipherService的encrypt即使用固定密钥对序列化内容加密

getIdentityToRemember函数中的rememberSerializedIdentity函数

```java
    protected void rememberSerializedIdentity(Subject subject, byte[] serialized) {
        if (!WebUtils.isHttp(subject)) {
            if (log.isDebugEnabled()) {
                String msg = "Subject argument is not an HTTP-aware instance.  This is required to obtain a servlet request and response in order to set the rememberMe cookie. Returning immediately and ignoring rememberMe operation.";
                log.debug(msg);
            }

        } else {
            HttpServletRequest request = WebUtils.getHttpRequest(subject);
            HttpServletResponse response = WebUtils.getHttpResponse(subject);
            String base64 = Base64.encodeToString(serialized);
            Cookie template = this.getCookie();
            Cookie cookie = new SimpleCookie(template);
            cookie.setValue(base64);
            cookie.saveTo(request, response);
        }
    }
```

将刚刚加密的内容base64编码，然后将cookie设为base64编码的内容，并与HttpRequest和HttpResponse绑定

##### 解密cookie

获取加密的rememberme

```java
protected PrincipalCollection getRememberedIdentity(SubjectContext subjectContext) {
    RememberMeManager rmm = getRememberMeManager();
    if (rmm != null) {
        try {
            return rmm.getRememberedPrincipals(subjectContext);
        } catch (Exception e) {
            if (log.isWarnEnabled()) {
                String msg = "Delegate RememberMeManager instance of type [" + rmm.getClass().getName() +
                        "] threw an exception during getRememberedPrincipals().";
                log.warn(msg, e);
            }
        }
    }
    return null;
}
```

getRememberedPrincipals

```java
    public PrincipalCollection getRememberedPrincipals(SubjectContext subjectContext) {
        PrincipalCollection principals = null;
        try {
            byte[] bytes = getRememberedSerializedIdentity(subjectContext);
            //SHIRO-138 - only call convertBytesToPrincipals if bytes exist:
            if (bytes != null && bytes.length > 0) {
                principals = convertBytesToPrincipals(bytes, subjectContext);
            }
        } catch (RuntimeException re) {
            principals = onRememberedPrincipalFailure(re, subjectContext);
        }

        return principals;
    }
```

getRememberedSerializedIdentity将subjectContext base64解码，然后对接吗后的内容调用convertBytesToPrincipals

```java
protected PrincipalCollection convertBytesToPrincipals(byte[] bytes, SubjectContext subjectContext) {
    if (getCipherService() != null) {
        bytes = decrypt(bytes);
    }
    return deserialize(bytes);
}
```

然后再decrypt函数aes解密，反序列化bytes

反序列化触发利用链实现rce

##### 利用
