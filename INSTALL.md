1. hareware and OS
2. file 
3. database
4. software
[root@mail ~]# mkdir /appconf
[root@mail ~]# mkdir /appdeploy
rsync -avvz --progress root@mail.xxx.com:/appconf/ .
rsync -avvz --progress root@mail.xxx.com:/appdeploy/ .


