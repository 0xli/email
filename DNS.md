### 邮件客户端相关参数
为了便于邮件客户端自动添加账号，DNS需要配置smtp,imap和pop的二级域名，比如smtp.callt.net,imap.callt.net,pop.callt.net
|  名称   | 二级域名 | 地址  | 端口 | 备注
|  ----  | ----  | ----  |----  | ----  |
| web  | |webmail.callt.net |9080,9443  | webmail服务器地址 |
| smtp  | smtp.callt.net |mail.callt.net |25，465| 发送邮件的服务器 |
| pop3  | pop.callt.net |mail.callt.net |1103,995| 邮件客户端的POP3服务器 |
| imap4  | imap.callt.net |mail.callt.net |143,993| 邮件客户端的IMAP服务器 |
### 有邮件相关的DNS设置相关参数
|  名称   | 域名或子域名 | 类型  | 值 | 备注
|  ----  | -----|----  |----  | ----  |
| MX  |   | MX | smtpgw.callt.net | 接收邮件的服务器 |
| spf  |    | txt | v=spf1 include:smtpgw.callt.net -all| 发送邮件的服务器 |
| dkim  | default._domainkey. | cname | default._domainkey.callt.net | 服务器签名 |

#### *域名解析：基础设置
邮件发送前，需要先进行域名解析，域名解析主要是发送方的域名和跟踪链接域名。
```
假设你的域名是xxx.com
a)  MX记录：smtpgw.callt.net
    txt记录："v=spf1 include:smtpgw.callt.net -all"
b)	Pop3：pop3.xxx.com, ip: mail.callt.net,端口：110；995
c)	Imap:imap.xxx.com, mail.callt.net,端口：143；993
d)	Smtp:smtp.xxx.com, mail.callt.net（用户邮件客户端的smtp服务器地址，与MX可不同）,端口：25或465
e)	Webmail:mail.xxx.com, ip: x.x.x.x, port：80,test:9080,9443
f)  DKIM: dig txt default._domainkey.callt.net
          https://help.aliyun.com/knowledge_detail/74626.html
实例：gfax.net
1. MX 记录：接收邮件的服务器地址
gfax.net mx smtpgw.callt.net
2. SPF 记录：发送邮件的服务器地址
gfax.net txt v=spf1 include:spf.callt.net -all
3. DKIM：发送服务器签名
default._domainkey.gfax.net   CNAME   default._domainkey.callt.net
邮件头的信息：
DKIM-Signature: v=1; a=rsa-sha256; q=dns/txt; c=simple/relaxed; t=1593585193;
	s=default; d=gfax.net; i=liwei@gfax.net;
	h=Content-Type:MIME-Version:Subject:Message-ID:To:From:Date; l=344;
	bh=r+4pFaYZ2sjADfpFXQKR934y8e+4melpZKpyN895Bic=;
	b=sH0adwt1ueYTVu8sWxoDgG50yA7jWPY5zyK7fAJC+7/sIHryPnZ4Xie7CImncg7x
	8+lEQc++oJfCSt2BTaa3dsN6f+ujBWCobb3YSQ1HBpGpty0mi9wEn6/+M1qdjSLMy5j
	3fJ+lSOuMdmu0wzz6Pm34ppG4kdmG+U4P1o7L4CE=
```
#### *域名解析：进阶，提高投递成功率
mx记录、spf记录、DKIM和DMARC记录
```
为了提供邮件的投递成功率，还需要在域名解析上进行进一步的配置，邮件投递服务采用上海亿业公司的邮件投递服务。
设置步骤为在贵司的根域名配置页面，对根域子域名添加相应的DNS记录：需要添加mx记录、spf记录、DKIM和DMARC记录，需要添加的记录值如下：
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
```
