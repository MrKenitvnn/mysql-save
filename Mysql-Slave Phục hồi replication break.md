### Sử dụng percona-xtrabackup
-- Dẫn link: https://kipalog.com/posts/Mot-cach-de-phuc-hoi-mysql-replication-break

##### backup database tren master
>sudo innobackupex  --user=megatron --password=optimus2771983  --parallel=8 /data/backupdb/

##### Vào folder backup vừa được tạo để lấy ra position
>[root@bemecloud 2019-06-17_10-51-22]# cat xtrabackup_binlog_info
>mysql-bin.000056	14263883

##### Tạo thư mục backup trên slave
>mkdir /data2/backup-2019-07/
>chown -R y:y /data2/backup-2019-07/

##### chuyển backup folder từ master sang slave
>rsync -azhPe "ssh -p 22222" /data/backupdb/2019-07-19_10-26-31/ y@10.148.0.3:/data2/backup-2019-07/

##### Apply backup tren slave
>sudo innobackupex  --use-memory=8G --apply-log /data2/backup-2019-07/

##### stop mysql 
>sudo service mysqld stop

##### Copy file master.info ra ngoài
>sudo cp /data2/mysql/master.info /data2/

##### Xoá nội dung datadir -- cẩn thận xoá nhầm nếu chưa config datadir ở /etc/my.cnf
>sudo rm -rf /data2/mysql/*

##### Restore db
>sudo innobackupex --copy-back --parallel=8 /data2/backup-2019-07/

##### Copy lại file master.info vào thư mục data dir
>cp /data2/master.info /data2/mysql/

##### Gán lại quyền
>sudo chown -R mysql:mysql /data2/mysql/<br/>
>find /data2/mysql -type d -exec chown -R mysql:mysql {} \;<br/>
>find /data2/mysql -type f -exec chown -R mysql:mysql {} \;<br/>

##### Khởi động lại slave
>sudo service mysqld start

##### Set lại master
>CHANGE MASTER TO<br/>
MASTER_HOST="sv1", <br/>
MASTER_PORT=3306, <br/>
MASTER_USER="replicatior", <br/>
MASTER_PASSWORD="Mật_Khẩu_Của_replicatior", <br/>
MASTER_LOG_FILE='mysql-bin.000056', MASTER_LOG_POS=14263883;<br/>

##### Chạy lại SLAVE
>START SLAVE;

