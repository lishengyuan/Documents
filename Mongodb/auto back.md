## Mongodb 定时备份

1. **新建备份目录/脚本文件**

    <pre>
    cd /usr/local/mongodb/
    mkdir backup
    cd /usr/local/mongodb/backup
    mkdir data log
    touch backup.sh
    chmod a+x backup.sh
    touch  remove_expire_data.sh
    chmod a+x  remove_expire_data.sh
    touch MongodbBackup.crontab
    </pre>
    
2. **编辑备份脚本**

    <pre>
    vi /usr/local/mongodb/backup/backup.sh
    </pre>

    按i键进入编辑模式

    Ctrl+c复制下面 2.1 处的脚本，然后右键把内容粘贴到backup.sh中

    按Esc键输入:wq保存退出
    

   2.1 备份脚本 (/usr/local/mongodb/backup/backup.sh)

        #!/bin/bash
        echo "backup start `date '+%Y-%m-%d %H:%M:%S'`"
        cd /usr/local/mongodb/backup/data
        foldername=`date '+%Y%m%d%H%M%S'`
        echo "mkdir ${foldername}"
        mkdir ${foldername}
        cd /usr/local/mongodb/bin
        echo "backup database"
        ./mongodump -u username -p password -d database_name -o /usr/local/mongodb/backup/data/
        ${foldername}
        echo "database backup done"
        echo "backup end `date '+%Y-%m-%d %H:%M:%S'`"


3. **编辑 删除过期备份文件脚本**
        <pre>
        vi /usr/local/mongodb/backup/remove_expire_data.sh
        </pre>

    按i键进入编辑模式
Ctrl+c复制下面 3.1 处的脚本，然后右键把内容粘贴到remove_expire_data.sh中
按Esc键输入:wq保存退出

    3.1 删除过期备份文件脚本(/usr/local/mongodb/backup/remove_expire_data.sh)

        #!/bin/bash
        backup_data_path="/usr/local/mongodb/backup/data/"
        expire_day=35

        today_date=`date '+%Y-%m-%d'`
        timestamp=$(date --date="$today_date" +%s);
        ((expire_timestamp=$timestamp - ($expire_day - 1) * 24 * 60 * 60))

        for dir in `ls ${backup_data_path}`
        do
        currentpath="${backup_data_path}${dir}"
        modifydata=`stat -c %Y ${currentpath}`

        if [ $modifydata -lt $expire_timestamp ]
        then
        echo "rm -r $currentpath"
        rm -r $currentpath
        fi

        done

4. **编辑定时任务脚本**

    <pre>
    vi /usr/local/mongodb/backup/MongodbBackup.crontab
    </pre>

   按i键进入编辑模式Ctrl+c复制下面 4.1 处的脚本，然后右键把内容粘贴到MongodbBackup.crontab中.按Esc键输入:wq保存退出

    4.1 定时任务脚本(/usr/local/mongodb/backup/MongodbBackup.crontab)
   <pre>
   0 0 * * * /usr/local/mongodb/backup/backup.sh >>/usr/local/mongodb/backup/log/$(date +"\%Y\%m\%d\%H\%M\%S").log 2>&1
   0 0 * * * /usr/local/mongodb/backup/remove_expire_data.sh >>/usr/local/mongodb/backup/log/remove_$(date +"\%Y\%m\%d\%H\%M\%S").log 2>&1
   </pre>

5. **把定时任务脚本添加到定时任务中**
   <pre>
   crontab /usr/local/mongodb/backup/MongodbBackup.crontab
   </pre>

6. **验证定时任务是否启动**
   <pre>
   crontab –l
   </pre>

   没有输出内容则说明没有定时任务，如输出4.1 脚本处的内容则说明定时任务已启动

