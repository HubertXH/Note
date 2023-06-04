#### 如下表所示，按第一列求和第二列

key | value
---|---
a | 10
b | 3
c | 5
a | 11
b | 8    


```
awk '{sum[$1]+=$2}END{for(c in sum){print c,sum[c]}}'
```
###### 若有多列需要统计则可以按如下进行追加

```
awk '{a[$1]+=$2}{b[$1]+=$3}END{for(c in a){print c,a[c],b[c]}}'
```


#### 统计MySql表中的行数

```
#!/bin/bash

source /etc/profile

currentFolder=`pwd`
echo "sh img_info_count.sh"
echo `date`

lockFile=$currentFolder/img_info_count.lock


if [  -f "$lockFile" ]; then
  echo "img_info_09 lock file exists. exit."
  exit 0;
fi



touch $lockFile

dbname=tableName
dbhost=***
dbusr=***
dbpwd=***

sleep=15

for i in {0..9}
do
  countFile=$currentFolder/table_info_$i.count

  touch $countFile

  echo "begin count for table table_info_$i"

  startTime=`date +'%s'`
  selectLastImageId="SELECT id FROM table_info_$i ORDER BY id DESC LIMIT 1;"
  lastImageId=`mysql -h$dbhost -u$dbusr -p$dbpwd --default-character-set=GBK $dbname -Ne "$selectLastImageId"`
  endTime=`date +'%s'`
  cost=$((endTime-startTime))
  echo "the last image id $lastImageId"
  
  startId=0

  while [$startId -le $lastImageId]
  do

    echo "begin get count for shopId=$shopId"

    endId=$((startId+10000))

    echo "startId=$startId, endId=$endId"

    selectCountSql="SELECT shop_id,COUNT(1) FROM table_info_$i WHERE id >= $startId AND id <$endId GROUP BY shop_id"

    echo $selectCountSql
    startTime=`date +'%s'`

    mysql -h$dbhost -u$dbusr -p$dbpwd --default-character-set=GBK $dbname -Ne "$selectCountSql" >> $countFile

    endTime=`date +'%s'`
    cost=$((endTime-startTime))
    echo "count for shopId=$shopId cost $cost s"
    startId=$endId
    echo "startId=$startId"
    sleep $sleep;
  done
  echo "end count for table table_info_$i"
done

#rm -rf $lockFile

echo "finished"
```

#### 文件同步

```
#!/bin/bash

iplist=(
)

for h in ${iplist[@]};

do

 echo "----------------$h------------------"
    scp -p sourceFile $h:targetFile

 echo "------------------------------------"

done

exit;

-----------------------------------------------

#!/bin/bash

CONF_FILE="hosts.conf"
if [ ! -f "$CONF_FILE" ]
then
    echo "No conf file found."
    exit 1
fi

DEPLOY_FILE="fileName"

for HOST in `cat $CONF_FILE`
do
    if [[ "$HOST" =~ ^\#.* ]]
    then
        echo "Skip $HOST"
        echo ""
        continue
    fi

    ssh -o ConnectTimeout=5 -q $HOST "echo ''" > /dev/null
    if [ $? -ne 0 ]
    then
        echo "HOST $HOST unreachable."
        echo ""
        continue
    fi

    echo $HOST
    ssh $HOST "mkdir -p /var/www; rm -rf /var/www/pp-supplier-provider/*"
    scp dist/$DEPLOY_FILE $HOST:/var/www/
    ssh $HOST "cd /var/www; unzip $DEPLOY_FILE -d pp-supplier-provider/"
    echo ""
done
echo "Done."

exit 0

```
##### 文件传输命令


```
scp -p sourceFile targetPath(host:/path)

rsync -aP $sourcefolder root@$h:$targetfolder --exclude bin  --exclude logs  --exclude conf --delete
```
