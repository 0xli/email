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
