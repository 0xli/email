```
假设你的域名是xxx.com
a)  MX记录：smtpgw.callt.net
    txt记录："v=spf1 include:smtpgw.callt.net -all"
b)	Pop3：pop3.xxx.com, ip: mail.callt.net,端口：110；995
c)	Imap:imap.xxx.com, mail.callt.net,端口：143；993
d)	Smtp:smtp.xxx.com, mail.callt.net（用户邮件客户端的smtp服务器地址，与MX可不同）,端口：25或435
e)	Webmail:mail.xxx.com, ip: x.x.x.x, port：80,test:9080,9443
```
