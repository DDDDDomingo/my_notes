# Linux查看日志命令

date 查看当前服务时间



tail -f text.log  实时刷新指定日志

tail -n 100 text.log 查看最后一百行日志

cat -n text.log 显示日志行数

cat -n text.log |grep "name"  过滤text.log日志中的name字段,并显示行数

cat -n text.log|tail -n +100|head -n 10 过滤当前日志100行后日志的前10行

可以在最后使用 more,less 进行翻页




top 查看当前活跃进程,占用资源等

top -H -p id 查看指定进程资源

pstree -p  查看当前所有进程和线程

pstree -p |wc -l   查看当前线程总数量



更多可以查看

菜鸟教程中的Linux 命令大全

<https://www.runoob.com/linux/linux-command-manual.html>

Linux命令手册

<http://linux.51yip.com/>