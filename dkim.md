### 公私钥对生成
```shell script
# 生成私钥
openssl genrsa -out dkim.pem 1024
# 转为 pkcs8
openssl pkcs8 -topk8 -nocrypt -in dkim.pem -outform der -out dkim.der
# 生成公钥
openssl rsa -in dkim.pem -pubout
```

### DNS 配置
增加一条`TXT`的解析记录，`selector._domainkey.domain`，将`selector`和`domain`换成相应的内容，
如：`default._domainkey.example.com`，解析内容如下，将`p`换成自己生成的公钥
```shell script
v=DKIM1;g=*;k=rsa;p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCf4lvV
llV2eoDqxartI0bUiJXDv+TVhFoGcheKocQyLGrTi8BKamhoDt8yKiecpCm1rZ/n
RyxSqIAJFMV3y/XslSVV2Sc48efPtrdViGUcGYNCC/KrqYNgCF7vRO2oAQ7ePPBo
hwcR1hzavGeY/AVxpEeIvixQNmunxkdaqHCLuQIDAQAB;s=email;t=s
```

### 代码修改
在原来的`eml`文件内加上`dkim`的签名，用到了第三方的工具 [java-utils-mail-dkim](https://github.com/markenwerk/java-utils-mail-dkim)
```java
public class SmtpClientService {

    private static DkimSigner DKIM_SIGNER;

    @Value("${dkim.private}")
    private String dkimPrivate;
    
    @Value("${dkim.domain}")
    private String dkimDomain;
    
    @Value("${dkim.selector}")
    private String dkimSelector;    

    private void createDkimSigner() throws Exception {
        if (DKIM_SIGNER == null) {
            DKIM_SIGNER = new DkimSigner(dkimDomain, dkimSelector, new File(dkimPrivate));
        }
    }

    private String dkimSign(String mailFrom, String emlPath) {
        try {
            logger.info("dkim: domain -> {}, selector -> {}, private key -> {}", dkimDomain, dkimSelector, dkimPrivate);
            createDkimSigner();

            Session session = Session.getInstance(System.getProperties());
            File emlFile = new File(emlPath);
            MimeMessage mimeMessage = new MimeMessage(session, new FileInputStream(emlFile));

            DKIM_SIGNER.setIdentity(mailFrom);
            DKIM_SIGNER.setHeaderCanonicalization(Canonicalization.SIMPLE);
            DKIM_SIGNER.setBodyCanonicalization(Canonicalization.RELAXED);
            DKIM_SIGNER.setSigningAlgorithm(SigningAlgorithm.SHA256_WITH_RSA);
            DKIM_SIGNER.setLengthParam(true);
            DKIM_SIGNER.setCopyHeaderFields(false);

            MimeMessage signMessage = new DkimMessage(mimeMessage, DKIM_SIGNER);
            String dkimPath = emlPath.replace(emlFile.getName(), emlFile.getName() + "_dkim");
            signMessage.writeTo(new FileOutputStream(new File(dkimPath)));

            logger.info("dkim sign success: {}", dkimPath);
            return dkimPath;
        } catch (Exception e) {
            logger.error("dkim sign eml error.");
            logger.error(e.getMessage(), e);
        }

        return emlPath;
    }
}
```

配置文件
```properties
dkim.private = /appconf/nsmtp/certificates/dkim.der
dkim.domain = callt.net
dkim.selector = default
```