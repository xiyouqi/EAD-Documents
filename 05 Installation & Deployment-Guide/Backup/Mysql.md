#### Windows 下 Mysql 备份

Windows 下 Mysql 备份有三种类型：

- 冷备（cold backup）：需要关mysql服务，读写请求均不允许状态下进行；
- 温备（warm backup）： 服务在线，但仅支持读请求，不允许写请求；
- 热备（hot backup）：备份的同时，业务不受影响。

除备份的类型以外，还要确定数据备份的数据范围，数据备份包括以下三种范围：

- 完全备份：full backup，备份全部字符集。
- 增量备份: incremental backup 上次完全备份或增量备份以来改变了的数据，不能单独使用，要借助完全备份，备份的频率取决于数据的更新频率。
- 差异备份：differential backup 上次完全备份以来改变了的数据。

此处只针对**热备 + 完全备份**方式的脚本进行解释说明；
以下是Windows 下 Mysql 备份的 bat 文件内容，根据每个数据库的设置修改 bat 文件，并加入到 Windows 计划任务中，实现 Mysql 的定时备份；

```
@echo off
rem ----------------------------------------------------------------------
rem This is the mysql backup script.
rem You can use this script to backup mysql data.
rem Author:peilongwu
rem Date:2015-12-21
rem ----------------------------------------------------------------------

rem GetDate
rem GetYear:%date:~0,4%GetMonth:%date:~5,2%:GetDay:%date:~8,2%
rem 定义当前日期为变量 
echo current date : %date%
set strYear=%date:~0,4%&set strMonth=%date:~5,2%&set strDay=%date:~8,2%
set strDate=%strYear%%strMonth%%strDay%

rem ----------------------------------------------------------------------
rem ----------------------Backup Start------------------------------------
echo ----------------------Backup Start-----------------------------------

rem 设置Mysql备份使用的变量
echo 设置Mysql备份使用的变量
rem 最大备份文件数

rem 设置数据库主目录
set BIN_HOME=C:\Program Files (x86)\MySQL\MySQL Server 5.5\
rem 设置数据库备份目录
set BACKUP_HOME=E:\mysql\
rem 设置数据库最大备份文件数
set MAXIMUM_BACKUP_FILES=10
rem 数据库备份目录名前缀
set BACKUP_FOLDERNAME=database_backup
rem mysql所在主机的主机名
set DB_HOSTNAME=localhost
rem mysql登录用户名
set DB_USERNAME=root
rem mysql登录密码
set DB_PASSWORD=test
rem 备份的数据库名，可以同时备份多个数据库，相互之间使用空格分割
set DATABASES=ead app

rem set mysql backup execute.
echo "Bash Database Backup Tool"         								
rem 数据库备份文件的目录名，根据备份前缀与日期生成
set BACKUP_FOLDER=%BACKUP_HOME%%BACKUP_FOLDERNAME%_%strDate%
rem 创建数据库备份文件目录
mkdir %BACKUP_FOLDER%

rem 统计需要被备份的数据库
set count=0
	for %%a in (%DATABASES%) do (
	   set /A count+=1
	)
echo "[+] %count% databases will be backuped..."
rem 循环这个数据库名称列表然后逐个备份这些数据库
rem 设置备份命令
set EXEC="%BIN_HOME%bin\mysqldump"
echo %EXEC%
echo %BACKUP_FOLDER%

for %%a in (%DATABASES%) do (
    echo " Mysql-Dumping: %%a"
    echo "----- Began------"
    echo %strDate%
    %EXEC% -h%DB_HOSTNAME% -u%DB_USERNAME% -p%DB_PASSWORD% %%a > "%BACKUP_FOLDER%\%%a.sql"
)

rem ----------------------------------------------------------------------
rem ----------------------Backup Remove------------------------------------
echo ----------------------Backup Remove-----------------------------------

rem Mysql备份文件删除，删除 7 天前的文件
rem 指定天数
set DaysAgo=10
  
>"%temp%/DstDate.vbs" echo LastDate=date()-%DaysAgo%
>>"%temp%/DstDate.vbs" echo FmtDate=right(year(LastDate),4) ^& right("0" ^& month(LastDate),2) ^& right("0" ^& day(LastDate),2)
>>"%temp%/DstDate.vbs" echo wscript.echo FmtDate
for /f %%a in ('cscript /nologo "%temp%/DstDate.vbs"') do (
  set "DstDate=%%a"
)
set DstDate=%DstDate:~0,4%%DstDate:~4,2%%DstDate:~6,2%

echo %DstDate%
set DEL_PATH=%BACKUP_HOME%%BACKUP_FOLDERNAME%_%DstDate%
  
echo Delete Local %DEL_PATH%
if exist %DEL_PATH% (rd /s /q %DEL_PATH%)

endlocal 

```