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

### 其他域 DNS 配置
主机记录 | 记录类型 | 记录值
---|---|---
@ | MX | mx.callt.net
@ | TXT | v=spf1 include:spf.callt.net -all
default._domainkey | CNAME | dkim.callt.net

### 代码修改
在原来的`eml`文件内加上`dkim`的签名，用到了第三方的工具 [java-utils-mail-dkim](https://github.com/markenwerk/java-utils-mail-dkim)
```java
public class SmtpClientService {

    private static DkimSigner DKIM_SIGNER;

    @Value("${dkim.private}")
    private String dkimPrivate;
    
    @Value("${dkim.selector}")
    private String dkimSelector;    

    private void createDkimSigner() throws Exception {
        if (DKIM_SIGNER == null) {
            DKIM_SIGNER = new DkimSigner(dkimDomain, dkimSelector, new File(dkimPrivate));
        }
    }

    private String dkimSign(String mailFrom, String emlPath) {
        try {
            String domain = mailFrom.substring(mailFrom.indexOf("@") + 1);
            logger.info("dkim: domain -> {}, selector -> {}, private key -> {}", domain, dkimSelector, dkimPrivate);
            DkimSigner dkimSigner = new DkimSigner(domain, dkimSelector, new File(dkimPrivate));
            Session session = Session.getInstance(System.getProperties());
            File emlFile = new File(emlPath);
            MimeMessage mimeMessage = new MimeMessage(session, new FileInputStream(emlFile));

            dkimSigner.setIdentity(mailFrom);
            dkimSigner.setHeaderCanonicalization(Canonicalization.SIMPLE);
            dkimSigner.setBodyCanonicalization(Canonicalization.RELAXED);
            dkimSigner.setSigningAlgorithm(SigningAlgorithm.SHA256_WITH_RSA);
            dkimSigner.setLengthParam(true);
            dkimSigner.setCopyHeaderFields(false);

            MimeMessage signMessage = new DkimMessage(mimeMessage, dkimSigner);
            String dkimPath = emlPath.replace(emlFile.getName(), emlFile.getName() + FileUtils.DKIM_SUFFIX);
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
dkim.selector = default
```

### SPF, DKIM 校验
1.添加依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<dependencies>
    <dependency>
        <groupId>com.sun.mail</groupId>
        <artifactId>javax.mail</artifactId>
        <version>1.6.2</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.apache.james.jspf/apache-jspf-resolver -->
    <dependency>
        <groupId>org.apache.james.jspf</groupId>
        <artifactId>apache-jspf-resolver</artifactId>
        <version>1.0.1</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.apache.james/james-server-mailet-dkim -->
    <dependency>
        <groupId>org.apache.james</groupId>
        <artifactId>james-server-mailet-dkim</artifactId>
        <version>3.4.0</version>
    </dependency>
</dependencies>
```

2.SPF代码
```java
public class TcpHandler extends IoHandlerAdapter {
    private static final SPF SPF = new DefaultSPF();
    private static final String SPF_PASS = "pass";
    
    private boolean checkSPF(String clientIp, String mailFrom) {
        try{
            String hostName = mailFrom.substring(mailFrom.indexOf("@") + 1);
            SPFResult spfResult = SPF.checkSPF(clientIp, mailFrom, hostName);
                
            return SPF_PASS.equalsIgnoreCase(spfResult.getResult());
        } catch (Exception e) {
            logger.error("spf check error.");
            ogger.error(e.getMessage(), e);
        }

        return false;
    }
}
```

3.DKIM代码

**MimeMessageHeaders.java**
```java
package com.allcom.toolkit;

import com.github.fge.lambdas.Throwing;
import com.github.steveash.guavate.Guavate;
import com.google.common.collect.ImmutableList;
import com.google.common.collect.ImmutableListMultimap;
import com.google.common.collect.Iterators;
import com.google.common.collect.Streams;
import org.apache.commons.lang3.tuple.Pair;
import org.apache.james.jdkim.api.Headers;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;
import java.util.List;
import java.util.Locale;

/**
 * @author lixianghuan@allcomchina.com
 * @date 2020/6/30 15:02
 * @extra code change the world
 * @description
 */
public class MimeMessageHeaders implements Headers {
    private final ImmutableListMultimap<String, String> headers;
    private final List<String> fields;

    public MimeMessageHeaders(MimeMessage message) throws MessagingException {
        ImmutableList<Pair<String, String>> headsAndLines = Streams.stream(Iterators.forEnumeration(message.getAllHeaderLines()))
                .map(Throwing.function(this::extractHeaderLine).sneakyThrow())
                .collect(Guavate.toImmutableList());

        fields = headsAndLines
                .stream()
                .map(Pair::getKey)
                .collect(Guavate.toImmutableList());

        headers = headsAndLines
                .stream()
                .collect(Guavate.toImmutableListMultimap(
                        pair -> pair.getKey().toLowerCase(Locale.US),
                        Pair::getValue));
    }

    public List<String> getFields() {
        return fields;
    }

    public List<String> getFields(String name) {
        return headers.get(name.toLowerCase(Locale.US));
    }

    private Pair<String, String> extractHeaderLine(String header) throws MessagingException {
        int fieldSeperatorPosition = header.indexOf(':');
        if (fieldSeperatorPosition <= 0) {
            throw new MessagingException("Bad header line: " + header);
        }
        return Pair.of(header.substring(0, fieldSeperatorPosition).trim(), header);
    }
}
```

**DkimVerifyUtil.java**
```java
package com.allcom.toolkit;

import org.apache.james.jdkim.DKIMVerifier;
import org.apache.james.jdkim.api.BodyHasher;
import org.apache.james.jdkim.api.Headers;
import org.apache.james.jdkim.api.SignatureRecord;
import org.apache.james.jdkim.exceptions.FailException;
import org.apache.james.jdkim.mailets.CRLFOutputStream;
import org.apache.james.jdkim.mailets.HeaderSkippingOutputStream;

import javax.mail.MessagingException;
import javax.mail.Session;
import javax.mail.internet.MimeMessage;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.util.List;
import java.util.Optional;

/**
 * @author lixianghuan@allcomchina.com
 * @date 2020/6/30 14:44
 * @extra code change the world
 * @description
 */
public class DkimVerifyUtil {

    private static final DKIMVerifier DKIM_VERIFIER = new DKIMVerifier();

    public static String verify(String emlPath, boolean forceCRLF) {
        StringBuilder sb = new StringBuilder();
        try {
            MimeMessage mimeMessage = new MimeMessage(Session.getDefaultInstance(System.getProperties()), new FileInputStream(emlPath));
            List<SignatureRecord> res = verify(mimeMessage, forceCRLF);

            if (res == null || res.isEmpty()) {
                // neutral
                sb.append("neutral (no signatures)");
            } else {
                // pass
                sb.append("pass");
                for (SignatureRecord rec : res) {
                    sb.append(" (");
                    sb.append("identity ");
                    sb.append(rec.getIdentity().toString());
                    sb.append(")");
                }
            }
        } catch (FailException fe) {
            fe.printStackTrace();
            // fail
            String relatedRecordIdentity = Optional.ofNullable(fe.getRelatedRecordIdentity())
                    .map(value -> "identity" + value + ":")
                    .orElse("");
            sb.append("fail (" + relatedRecordIdentity + fe.getMessage() + ")");
        } catch (Exception e) {
            e.printStackTrace();
            sb.append("fail (" + e.getMessage() + ")");
        }

        return sb.toString();
    }

    private static List<SignatureRecord> verify(MimeMessage message, boolean forceCRLF) throws MessagingException, FailException {
        Headers headers = new MimeMessageHeaders(message);
        BodyHasher bh = DKIM_VERIFIER.newBodyHasher(headers);
        try {
            if (bh != null) {
                OutputStream os = new HeaderSkippingOutputStream(bh
                        .getOutputStream());
                if (forceCRLF) {
                    os = new CRLFOutputStream(os);
                }
                message.writeTo(os);
            }

        } catch (IOException e) {
            throw new MessagingException("Exception calculating bodyhash: "
                    + e.getMessage(), e);
        } finally {
            try {
                if (bh != null) {
                    bh.getOutputStream().close();
                }
            } catch (IOException e) {
                throw new MessagingException("Exception calculating bodyhash: "
                        + e.getMessage(), e);
            }
        }
        return DKIM_VERIFIER.verify(bh);
    }

}
```

```java
public class TcpHandler extends IoHandlerAdapter {
    private static final SPF SPF = new DefaultSPF();
    private static final String DKIM_PASS = "pass";
    
    private boolean checkDKIM(String emlPath, boolean forceCRLF) {
        try {
            String dkimResult = DkimVerifyUtil.verify(emlPath, true);
            logger.info("dkim verify result: {}", dkimResult);

            return dkimResult.startsWith(DKIM_PASS);
        } catch (Exception e) {
            logger.error("check dkim error.");
            logger.error(e.getMessage(), e);
        }

        return false;
    }
}
```