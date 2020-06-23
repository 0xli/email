1. hareware and OS
```
db: 512G, mysql, authcenter
mail: 1T SSD，smtp,pop3,imap
webmail: newwebmail,mailapi
smtwgw: rbl, smtwgw,smtpgwmanager,mailadmin
```
2. file 
```
2.1 db:格式化和加载/dev/sdb(512G)到/data
2.2 mail:格式化和加载/dev/sdb（1T）到/data
```
3. database
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
4. software
```
[root@mail ~]# mkdir /appconf
[root@mail ~]# mkdir /appdeploy
rsync -avvz --progress /appconf/ root@mail.xxx.com:/appconf
rsync -avvz --progress /appdeploy/ root@mail.xxx.com:/appdeploy
````

