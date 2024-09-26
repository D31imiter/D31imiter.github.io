#### Shiro-1.2.4  CVE-2016-4437

##### 加密

```
 public void onSuccessfulLogin(Subject subject, AuthenticationToken token, AuthenticationInfo info) {
        //清理旧的身份验证信息c
        forgetIdentity(subject);

        //生成新的身份验证信息
        if (isRememberMe(token)) {  //如果有勾选remember me
            rememberIdentity(subject, token, info);//生成新的cookie中的RememberMe字段
```

如果勾选了RememberMe选项就会调用rememberIdentity函数

```
 public void rememberIdentity(Subject subject, AuthenticationToken token, AuthenticationInfo authcInfo) {
    PrincipalCollection principals = getIdentityToRemember(subject, authcInfo);//获取身份信息，比如root，并赋值到principals
    rememberIdentity(subject, principals);//继续跟进rememberIdentity
 }
```

```
protected void rememberIdentity(Subject subject, PrincipalCollection accountPrincipals) {
    byte[] bytes = convertPrincipalsToBytes(accountPrincipals);//将账户主体（root）信息转换为字节
    rememberSerializedIdentity(subject, bytes);
}
```

跟进convertPrincipalsToBytes函数:

```
    protected byte[] convertPrincipalsToBytes(PrincipalCollection principals) {
        byte[] bytes = serialize(principals);//使用serialize()序列化身份信息（root）
        if (getCipherService() != null) {
            bytes = encrypt(bytes);//加密序列化后的身份信息
        }
        return bytes;
    }
```

跟进加密方式encrypt：

```
    protected byte[] encrypt(byte[] serialized) {
        byte[] value = serialized;
        CipherService cipherService = getCipherService();   //使用getCipherService()获取加密方式，并赋值到cipherService
        if (cipherService != null) {    //如果cipherService不为空
            ByteSource byteSource = cipherService.encrypt(serialized, getEncryptionCipherKey());    //对序列化的身份信息进行cipherService
							  				    方式的加密,getEncryptionCipherKey()为加密密钥
            value = byteSource.getBytes();
        }
        return value;
    }
```

用getEncryptionCipherKey()获取的密钥对serialized加密，跟进密钥：是一个常量，kPH+bIxk5D2deZiIxcaaaA==

跟进刚刚的rememberSerializedIdentity：

```
    protected void rememberSerializedIdentity(Subject subject, byte[] serialized) {

        if (!WebUtils.isHttp(subject)) {
            if (log.isDebugEnabled()) {
                String msg = "Subject argument is not an HTTP-aware instance.  This is required to obtain a servlet " +
                        "request and response in order to set the rememberMe cookie. Returning immediately and " +
                        "ignoring rememberMe operation.";
                log.debug(msg);
            }
            return;
        }


        HttpServletRequest request = WebUtils.getHttpRequest(subject);
        HttpServletResponse response = WebUtils.getHttpResponse(subject);

        //base 64 encode it and store as a cookie:
        String base64 = Base64.encodeToString(serialized);  //进行base64编码

        Cookie template = getCookie(); //the class attribute is really a template for the outgoing cookies
        Cookie cookie = new SimpleCookie(template);
        cookie.setValue(base64);    //将base64编码的信息整合到cookie当中
        cookie.saveTo(request, response);
    }
```

cookie生成流程大致为：序列化身份信息root-->再根据已知密钥（kPH+bIxk5D2deZiIxcaaaA==）进行AES加密-->base64编码-->生成cookie信息

##### 解密

```
    public PrincipalCollection getRememberedPrincipals(SubjectContext subjectContext) {
        PrincipalCollection principals = null;
        try {
            byte[] bytes = getRememberedSerializedIdentity(subjectContext); //提取cookie，并对其进行base64解码
            //SHIRO-138 - only call convertBytesToPrincipals if bytes exist:
            if (bytes != null && bytes.length > 0) {
                principals = convertBytesToPrincipals(bytes, subjectContext);
            }   //进行AES解密与反序列化处理
        } catch (RuntimeException re) {
            principals = onRememberedPrincipalFailure(re, subjectContext);
        }

        return principals;
    }
```

跟进convertBytesToPrincipals：

```
    protected PrincipalCollection convertBytesToPrincipals(byte[] bytes, SubjectContext subjectContext) {
        if (getCipherService() != null) {
            bytes = decrypt(bytes); //AES解密
        }
        return deserialize(bytes);  //反序列化操作
    }
```

在decrypt()处断点跟进：

```
    protected byte[] decrypt(byte[] encrypted) {
        byte[] serialized = encrypted;
        CipherService cipherService = getCipherService();
        if (cipherService != null) {
            ByteSourceBroker broker = cipherService.decrypt(encrypted, getDecryptionCipherKey());
            serialized = broker.getClonedBytes();
        }
        return serialized;
    }
```

固定密钥解密，导致反序列化漏洞

### 测试案例

```
    public PrincipalCollection getRememberedPrincipals(SubjectContext subjectContext) {//subjectContext 为传入的cookie值
        PrincipalCollection principals = null;
        byte[] bytes = null;
        try {
            bytes = getRememberedSerializedIdentity(subjectContext);		//Base64解码
            //SHIRO-138 - only call convertBytesToPrincipals if bytes exist:
            if (bytes != null && bytes.length > 0) {
                principals = convertBytesToPrincipals(bytes, subjectContext);	//原始数据等于AES解密后的Cookie
            }
        } catch (RuntimeException re) {
            principals = onRememberedPrincipalFailure(re, subjectContext);
        } finally {
            ByteUtils.wipe(bytes);
        }

        return principals;
    }
```

principals等于解密后的字符串，找到哪里调用了getRememberedPrincipals函数

```
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

```
    public void testGetRememberedPrincipalsWithEmptySerializedBytes() {
        AbstractRememberMeManager rmm = new DummyRememberMeManager();
        //Since the dummy's getRememberedSerializedIdentity implementation returns an empty byte
        //array, we should be ok:
        PrincipalCollection principals = rmm.getRememberedPrincipals(new DefaultSubjectContext());
        assertNull(principals);

        //try with a null return value too:
        rmm = new DummyRememberMeManager() {
            @Override
            protected byte[] getRememberedSerializedIdentity(SubjectContext subjectContext) {
                return null;
            }
        };
        principals = rmm.getRememberedPrincipals(new DefaultSubjectContext());
        assertNull(principals);
    }
```
