## 以下是执行数据库备份的脚本
- /root/myshelldir/mysqlshell.sh
```shell
#执行此文件时的时间(注意这里的格式转换)
BACKUPTIME=$(date +%Y-%m-%d_%H%M%S)

#数据库地址
HOST=39.103.200.101

#数据库用户名
USER=root

#数据库密码
PASSWORD=12345678

#需要备份的数据库名称
DATABASE=health


#创建备份目录，如果存在，则不用创建(注意这里如何递归创建目录)
[ ! -d "$BACKUPPATH/$BACKUPTIME" ]&& mkdir -p "$BACKUPPATH/$BACKUPTIME"

#备份数据库（注意这里如何进行数据库的配置的）
mysqldump -u$USER -p$PASSWORD --host=$HOST -q -R --databases $DATABASE | gzip> $BACKUPPATH/$BACKUPTIME/$BACKUPTIME.sql.gz

#将文件处理成tar.gz
cd ${BACKUPPATH}
tar -zcvf $BACKUPTIME.tar.gz  ${BACKUPTIME}
rm -rf ${BACKUPPATH}/${BACKUPTIME}

#删除10天前的备份文件
find ${BACKUPPATH} -atime +10 -name ".tar.gz" -exec rm -rt {} \;
echo "备份数据库${DATABASE}成功"
```

- 上面关于`find -exec` 的解释:
> exec解释：
> -exec 参数后面跟的是command命令，它的终止是以;为结束标志的，所以这句命令后面的分号是不可缺少的，考虑到各个系统中分号会有不同的意义，所以前面加反斜杠。
> {} 花括号代表前面find查找出来的文件名。
> 使用find时，只要把想要的操作写在一个文件里，就可以用exec来配合find查找，很方便的。在有些操作系统中只允许-exec选项执行诸如l s或ls -l这样的命令。大多数用户使用这一选项是为了查找旧文件并删> 除它们。建议在真正执行rm命令删除文件之前，最好先用ls命令看一下，确认它们是所要删除的文件。 exec选项后面跟随着所要执行的命令或脚本，然后是一对儿{ }，一个空格和一个\，最后是一个分号。为了> > 使用exec选项，必须要同时使用print选项。如果验证一下find命令，会发现该命令只输出从当前路径起的相对路径及文件名。

- 上面关于`find -atime +10` 的解释:
> 表示文件在10天以上没有被访问过的文件
[-atime参考文章](https://blog.csdn.net/wodeqingtian1234/article/details/53975744?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164639461616780264023449%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=164639461616780264023449&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-1-53975744.pc_search_result_cache&utm_term=-atime&spm=1018.2226.3001.4187)

所以find所在行的意思就是找出10天以上没有访问过且以.tar.gz结尾的文件，将他们递归分询问删除

## 下面编写定时文件脚本：
输入命令：`crontab -e`进入编辑界面
输入：

![image](https://user-images.githubusercontent.com/85269099/156760096-5d91b32f-b2ec-457d-884d-190ff0323939.png)

至此：现在已经开启数据库中health库的自动备份，每天2：30自动备份，且会自动删除10天前的备份数据，防止linux硬盘压力过大。



