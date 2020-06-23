1. hareware and OS
2. file 
2.1 db:格式化和加载/dev/sdb(512G)到/data
2.2 mail:格式化和加载/dev/sdb(1T)到/data
3. database
4. software
```
[root@mail ~]# mkdir /appconf
[root@mail ~]# mkdir /appdeploy
rsync -avvz --progress /appconf/ root@mail.xxx.com:/appconf
rsync -avvz --progress /appdeploy/ root@mail.xxx.com:/appdeploy
````

