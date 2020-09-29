### PORT 25, 587
```
https://www.mailgun.com/blog/which-smtp-port-understanding-ports-25-465-587/
https://pepipost.com/blog/25-465-587-2525-choose-the-right-smtp-port/
```
### 关于邮件投递失败的处理
```
1、判断是否需要重新投递；
2、放弃重新投递后，webmail显示失败及失败的详细信息，给用户回一封投递失败及其原因的邮件（即退信，主意避免产生过多的退信）；
3、mailadmin界面里可以看到投递失败的信息及统计；
4、通过亿业的邮件投递服务再次发送，由于目前亿业需要接管MX记录，只能通过二级域名进行投递，
   为每个用户创建一个对应二级域名的地址，比如lxh@callt.net对应lxh@mail.callt.net, 以二级域名对应的地址投递，
   replyto设为原始邮件地址；
5、与亿业探讨二级域名的mx的问题   
```
### 自动转发
sql = "insert into autoforward(umid,forwardto) values(?,?)";

### 转发记录
webmail公用户端查询:
1. 转发的邮件已经投递状态在发件箱中显示，目前收件箱转发的邮件是否记录到了数据库，如果没有需要记录，用户查询时可以触发查询亿业邮件投递结果并以此作为邮件的最终投递状态和结果；

### 收发记录
为了解决邮件投递的互通性需要往来邮件的记录，可以只记录一天或者72小时；
smtpgw和smtp管理界面里都需要显示收发邮件的状态，需要了解是否在数据库中有记录，没有的话要加上记录；
