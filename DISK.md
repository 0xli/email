```
1. fdisk -l
2. format using mkfs
   mkfs -t ext4 -c /dev/sdb
3. mount 
   mount  /dev/sdb /data
4 vi /etc/fstab --修改启动自动挂载项
/dev/sdb /data ext4 defaults 0 0   
5. df -lhT
```
https://blog.csdn.net/zwl18210851801/article/details/81477862
