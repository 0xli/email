### 域名解析
邮件发送前，需要先进行域名解析，域名解析主要是发送方的域名和跟踪链接域名。
```
假设你的域名是xxx.com
a)  MX记录：smtpgw.callt.net
    txt记录："v=spf1 include:smtpgw.callt.net -all"
b)	Pop3：pop3.xxx.com, ip: mail.callt.net,端口：110；995
c)	Imap:imap.xxx.com, mail.callt.net,端口：143；993
d)	Smtp:smtp.xxx.com, mail.callt.net（用户邮件客户端的smtp服务器地址，与MX可不同）,端口：25或435
e)	Webmail:mail.xxx.com, ip: x.x.x.x, port：80,test:9080,9443
```
### mx记录、spf记录、DKIM和DMARC记录
设置步骤为在贵司的根域名配置页面，对子域名添加相应的DNS记录：需要添加mx记录、spf记录、DKIM和DMARC记录，需要添加的记录值如下：
MX记录：mx8.gmail.easeye.com.cn
txt记录："v=spf1 include:easeye-edm.com -all"
DKIM记录：easeye2007._domainkey.easeye.com.cn.
DMARC记录: _dmarc.easeye.com.cn.
以newsletter.XXX.com为例，需要添加的记录值如下：
newsletter.XXX.com mx mx8.gmail.easeye.com.cn
newsletter.XXX.com txt "v=spf1 mx a include:easeye-edm.com -all"
在“newsletter.XXX.com.”域名下新增两个子域名并设置相应的cname记录如下：
easeye2007._domainkey.newsletter.XXX.com. cname easeye2007._domainkey.easeye.com.cn.
_dmarc.newsletter.XXX.com. cname _dmarc.easeye.com.cn.
