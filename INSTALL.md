1.hareware and OS
```
db: 512G, mysql, authcenter
mail: 1T SSD，smtp,pop3,imap
webmail: newwebmail,mailapi
smtwgw: rbl, smtwgw,smtpgwmanager,mailadmin

nameserver 114.114.114.114
keep dns configure:
chattr +i /etc/resolv.conf

```

2.file 
```
2.1 db:格式化和加载/dev/sdb(512G)到/data
2.2 mail:格式化和加载/dev/sdb（1T）到/data
```

3.database
```shell script
# 下载数据库 sql 文件
scp -r root@139.129.193.117:/data/dbbak/mail-sql .
# 创建数据库
docker exec -i mysql-5.7 mysql -uroot -pallcom -e "create database aum"
docker exec -i mysql-5.7 mysql -uroot -pallcom -e "create database aumqyyj"
docker exec -i mysql-5.7 mysql -uroot -pallcom -e "create database aumright"
docker exec -i mysql-5.7 mysql -uroot -pallcom -e "create database authlog"
docker exec -i mysql-5.7 mysql -uroot -pallcom -e "create database smtplog"
docker exec -i mysql-5.7 mysql -uroot -pallcom -e "create database smtpdb"
docker exec -i mysql-5.7 mysql -uroot -pallcom -e "create database aumrbllist"

# 导入数据库
 docker exec -i mysql-5.7 mysql -uroot -pallcom aum < aum.20200623.sql
 docker exec -i mysql-5.7 mysql -uroot -pallcom aumqyyj < aumqyyj.20200623.sql
 docker exec -i mysql-5.7 mysql -uroot -pallcom aumright < aumright.20200623.sql
 docker exec -i mysql-5.7 mysql -uroot -pallcom authlog < authlog.20200623.sql
 docker exec -i mysql-5.7 mysql -uroot -pallcom smtplog < smtplog.20200623.sql
 docker exec -i mysql-5.7 mysql -uroot -pallcom smtpdb < smtpdb.20200623.sql
 docker exec -i mysql-5.7 mysql -uroot -pallcom aumrbllist < aumrbllist.20200623.sql
```

4.software
```
[root@mail ~]# mkdir /appconf
[root@mail ~]# mkdir /appdeploy
rsync -avvz --progress /appconf/ root@mail.xxx.com:/appconf
rsync -avvz --progress /appdeploy/ root@mail.xxx.com:/appdeploy
````

5.data

程序正常运行需要在数据库中如下初始化数据
```mysql
INSERT INTO `aumqyyj`.`mailsystem` VALUES 
('allcomchina.com',md5('allcom5131'),null,'','allcomchina.com.gif','','_index_10.jsp',314572800000,200,'','','','','',now(),1),
('callt.net',md5('allcom5131'),null,'','','allcom voip','',10737418240,1000,'','','','','',now(),0);
INSERT INTO `aumqyyj`.`systemparam` VALUES 
(NULL,'domain','','allcomchina.com'),
(NULL,'faxdbip','','10.10.0.36'),
(NULL,'faxdbuser','','root'),
(NULL,'faxdbpass','','allcom'),
(NULL,'maxstore','','52428800'),
(NULL,'maxtag','','50'),
(NULL,'mergesend','','merge@gfax.cn'),
(NULL,'multiaccount','','false'),
(NULL,'oooip','','10.10.0.112'),
(NULL,'oooport','','2002'),
(NULL,'outfax','','outfax@gfax.cn'),
(NULL,'outfaxlist','','outfaxb@gfax.cn'),
(NULL,'outresend','','outfax@gfax.cn'),
(NULL,'outsms','','sms@gfax.cn'),
(NULL,'queue','','/data/smtp/queue'),
(NULL,'smtpip','','10.12.28.12'),
(NULL,'smtpport','','25'),
(NULL,'spambox','','spam@mailwalk.com'),
(NULL,'version','','fax'),
(NULL,'isloaddirectory','','true'),
(NULL,'archivport','','2050'),
(NULL,'lasthash','lasthash',''),
(NULL,'directorydbpass','','allcom'),
(NULL,'filestore','','/data/filestore'),
(NULL,'archivip','','10.10.0.202'),
(NULL,'spam','reportspampath','/data/smtp/spam'),
(NULL,'ham','reporthampath','/data/smtp/ham'),
(NULL,'issessionURL','','http://10.10.0.42:8080/rest/service/issessionid/'),
(NULL,'maxrcptto','maxrcptto','100'),
(NULL,'directorydbip','','10.12.28.10'),
(NULL,'directorydbuser','','root'),
('callt.net','maildir','','/data/email/callt.net'),
('callt.net','tableindex','tableindex','0');
INSERT INTO `smtpdb`.`getmailtask_control` (`getstatus`, `lasttime`) VALUES ('0', now());
INSERT INTO `smtpdb`.`getparsetask_control` (`getstatus`, `lasttime`) VALUES ('0', now());
INSERT INTO `aumqyyj`.`getmailtask_control` (`getstatus`, `lasttime`) VALUES ('0', now());
INSERT INTO `aumqyyj`.`getparsetask_control` (`getstatus`, `lasttime`) VALUES ('0', now());
INSERT INTO `aumright`.`rcpthosts` VALUES ('callt.net','10.12.28.12',25,1);
INSERT INTO `aumright`.`allowinlist` VALUES ('liwei@callt.net'),('lxh@callt.net');
INSERT INTO `aumright`.`gateway_manager` VALUES ('admin',md5('allcom5131'));
```

除此之外，还需要注意配置文件中的配置
```properties
# smtp
# 网关的ip必须加上
system.relayip = 10.12.28.13
```