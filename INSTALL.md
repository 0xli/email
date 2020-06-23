1. hareware and OS
2. file 
3. database
4. software
```
[root@mail ~]# mkdir /appconf
[root@mail ~]# mkdir /appdeploy
rsync -avvz --progress /appconf/ root@mail.xxx.com:/appconf
rsync -avvz --progress /appdeploy/ root@mail.xxx.com:/appdeploy
````

