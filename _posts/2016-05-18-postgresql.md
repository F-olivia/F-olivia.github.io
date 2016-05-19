---
layout: post
title: "PostgreSQL学习系列（二）"
description: "postgres的源码安装-第三方插件安装"
category: "database"
tags: [database,postgresql]
---
{% include JB/setup %}

<ul>
    <li>作者：<a href="http://weibo.com/Polivia" target="blank">F-olivia</a></li>
    <li>本文地址：http://f-olivia.github.io/database/2016/05/18/postgresql/</li>
    <li>转载请注明出处</li>
</ul>

##### 源码包中自带的插件包安装
在源码安装时，用gmake install，应该是自带的所有插件包都已编译，
gmake world和gmake install-world 所有的插件都会被安装, 那么就不需要再次安装了.在源码包目录/contrib下，

 1.安装所有的插件包：只要执行：gmake && gmake install ，

 2.安装单个插件包：那就进入到所在插件包目录，比如pg_stat_statements，
	
	[root@localhost contrib]# cd pg_stat_statements/
	[root@localhost pg_stat_statements]# ll
	total 476
	-rw-r--r--. 1 1107 1107    687 Mar 29 04:07 Makefile
	-rw-r--r--. 1 1107 1107   1246 Mar 29 04:07 pg_stat_statements--1.0--1.1.sql
	-rw-r--r--. 1 1107 1107   1336 Mar 29 04:07 pg_stat_statements--1.1--1.2.sql
	-rw-r--r--. 1 1107 1107   1454 Mar 29 04:07 pg_stat_statements--1.2--1.3.sql
	-rw-r--r--. 1 1107 1107   1399 Mar 29 04:07 pg_stat_statements--1.3.sql
	-rw-r--r--. 1 1107 1107    449 Mar 29 04:07 pg_stat_statements--unpackaged--1.0.sql
	-rw-r--r--. 1 1107 1107  87326 Mar 29 04:07 pg_stat_statements.c
	-rw-r--r--. 1 1107 1107    191 Mar 29 04:07 pg_stat_statements.control
	-rw-r--r--. 1 root root 223496 May 14 06:41 pg_stat_statements.o
	-rwxr-xr-x. 1 root root 143100 May 14 06:41 pg_stat_statements.so
	[root@localhost pg_stat_statements]# make install

 3.使用插件包，还拿pg_stat_statements来说，
	
	---1)到数据库中以超级用户执行create extension
	postgres=# create extension pg_stat_statements;
	CREATE EXTENSION
	---2)到配置参数文件中增加参数值
	postgres@localhost-> vi postgresql.conf 
	shared_preload_libraries = 'pg_stat_statements' 
	pg_stat_statements.max = 10000
	pg_stat_statements.track = all
	....
	---3)重启pg，测试
	postgres@localhost-> pg_ctl restart -m fast
	waiting for server to shut down.... done
	postgres@localhost-> psql
	psql (9.5.2)
	Type "help" for help.
	
	postgres=# select * from pg_stat_statements;
	 userid | dbid | queryid | query | calls | total_time | min_time | max_time | 
	mean_time | stddev_time | rows | shared_blks_hit | shared_blks_read | shared_b
	lks_dirtied | shared_blks_written | local_blks_hit | local_blks_read | local_b
	lks_dirtied | local_blks_written | temp_blks_read | temp_blks_written | blk_re
	ad_time | blk_write_time 
	--------+------+---------+-------+-------+------------+----------+----------+-
	----------+-------------+------+-----------------+------------------+---------
	------------+---------------------+----------------+-----------------+--------
	------------+--------------------+----------------+-------------------+-------
	--------+----------------
	(0 rows)
	
	postgres=# select * from pg_stat_statements;
	 userid | dbid  |  queryid   |               query               | calls | tot
	al_time | min_time | max_time | mean_time | stddev_time | rows | shared_blks_h
	it | shared_blks_read | shared_blks_dirtied | shared_blks_written | local_blks
	_hit | local_blks_read | local_blks_dirtied | local_blks_written | temp_blks_r
	ead | temp_blks_written | blk_read_time | blk_write_time 
	--------+-------+------------+-----------------------------------+-------+----
	--------+----------+----------+-----------+-------------+------+--------------
	---+------------------+---------------------+---------------------+-----------
	-----+-----------------+--------------------+--------------------+------------
	----+-------------------+---------------+----------------
	     10 | 13252 | 4291696289 | select * from pg_stat_statements; |     1 |    
	  1.012 |    1.012 |    1.012 |     1.012 |           0 |    0 |              
	 0 |                0 |                   0 |                   0 |           
	   0 |               0 |                  0 |                  0 |            
	  0 |                 0 |             0 |              0
	(1 row)

##### 第三方插件安装

根据第三方插件提供的安装说明进行安装
通用的安装方法，以安装debug包为例

1. 把第三方插件的源码目录拷贝到contrib目录中
	---首先，下载安装包pldebugger，步骤：
	1）打开网页http://git.postgresql.org/，找到安装包如图1：

![Alt text](/assets/blog-images/20160519100642.png)
	
	2）点击snapshot，如图2：

![Alt text](/assets/blog-images/20160519101009.png)

	3）把安装包拷贝到contrib目录中
	[root@localhost ~]# ls
	anaconda-ks.cfg            postgresql-9.5.2
	pldebugger-14c6caf.tar.gz  postgresql-9.5.2.tar.bz2
	[root@localhost ~]# tar -zxvf pldebugger-14c6caf.tar.gz 
	pldebugger-14c6caf/
	......
	pldebugger-14c6caf/settings.projinc
	pldebugger-14c6caf/uninstall_pldbgapi.sql
	[root@localhost ~]# ll
	total 18076
	-rw-------. 1 root root     1089 May 12 00:28 anaconda-ks.cfg
	drwxrwxr-x. 2 root root     4096 Mar  1 05:33 pldebugger-14c6caf
	-rw-r--r--. 1 root root    47764 May 19 00:17 pldebugger-14c6caf.tar.gz
	drwxrwxrwx. 6 1107 1107     4096 May 14 06:36 postgresql-9.5.2
	-rw-r--r--. 1 root root 18446616 May 13 17:56 postgresql-9.5.2.tar.bz2
	[root@localhost ~]# cd pldebugger-14c6caf/
	[root@localhost pldebugger-14c6caf]# ll
	total 212
	-rw-rw-r--. 1 root root  2585 Mar  1 05:33 Makefile
	-rw-rw-r--. 1 root root  3711 Mar  1 05:33 README.pldebugger
	......
	-rw-rw-r--. 1 root root  1387 Mar  1 05:33 uninstall_pldbgapi.sql
	[root@localhost pldebugger-14c6caf]# vi README.pldebugger 
	PostgreSQL pl/pgsql Debugger API
	================================
	
	This module is a set of shared libraries which implement an API for debugging
	pl/pgsql functions on PostgreSQL 8.4 and above. The pgAdmin III project
	(http://www.pgadmin.org/) provides a client user interface as part of pgAdmin
	III v1.10.0 and above.
	
	If you wish to debug functions on PostgreSQL 8.2 or 8.3, please use v0.93 of
	the debugger plugin, or checkout the PRE_8_4_SERVER branch from CVS.
	
	
	Installation
	------------
	
	- Copy this directory to contrib/ in your PostgreSQL source tree.
	
	- Run 'make; make install'
	
	- Edit your postgresql.conf file, and modify the shared_preload_libraries config
	  option to look like:
	
	  shared_preload_libraries = '$libdir/plugin_debugger'
	
	- Restart PostgreSQL for the new setting to take effect.
	
	- Run the following command in the database or databases that you wish to
	  debug functions in:
	
	  CREATE EXTENSION pldbgapi;
	
	  (on server versions older than 9.1, you must instead run the pldbgapi--1.0.sql
	  script directly using psql).
	[root@localhost ~]# mv pldebugger-14c6caf postgresql-9.5.2/contrib/
	

2. 把pg_config加入到PATH中,其实就是上面到配置参数文件中增加参数值
	
	[root@localhost lib]# su - postgres
	Last login: Wed May 18 16:59:05 CST 2016 on pts/0
	postgres@localhost-> cd $PGDATA
	postgres@localhost-> vi postgresql.conf 
	shared_preload_libraries = 'pg_stat_statements,plugin_debugger' 

3. gmake clean; gmake; gmake install
	
	[root@localhost ~]# cd postgresql-9.5.2/contrib/pldebugger-14c6caf/
	[root@localhost pldebugger-14c6caf]# make
	gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -g -O2 -fpic -I../../src/pl/plpgsql/src -I. -I. -I../../src/include -D_GNU_SOURCE -I/usr/include/libxml2   -c -o plpgsql_debugger.o plpgsql_debugger.c
	gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -g -O2 -fpic -I. -I. -I../../src/include -D_GNU_SOURCE -I/usr/include/libxml2   -c -o plugin_debugger.o plugin_debugger.c
	gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -g -O2 -fpic -I. -I. -I../../src/include -D_GNU_SOURCE -I/usr/include/libxml2   -c -o dbgcomm.o dbgcomm.c
	gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -g -O2 -fpic -I. -I. -I../../src/include -D_GNU_SOURCE -I/usr/include/libxml2   -c -o pldbgapi.o pldbgapi.c
	gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -g -O2 -fpic -shared -o plugin_debugger.so plpgsql_debugger.o plugin_debugger.o dbgcomm.o pldbgapi.o -L../../src/port -L../../src/common -Wl,--as-needed -Wl,-rpath,'/opt/pgsql9.5.2/lib',--enable-new-dtags  
	[root@localhost pldebugger-14c6caf]# make install
	/usr/bin/mkdir -p '/opt/pgsql9.5.2/lib'
	/usr/bin/mkdir -p '/opt/pgsql9.5.2/share/extension'
	/usr/bin/mkdir -p '/opt/pgsql9.5.2/share/extension'
	/usr/bin/mkdir -p '/opt/pgsql9.5.2/share/doc//extension'
	/usr/bin/install -c -m 755  plugin_debugger.so '/opt/pgsql9.5.2/lib/plugin_debugger.so'
	/usr/bin/install -c -m 644 ./pldbgapi.control '/opt/pgsql9.5.2/share/extension/'
	/usr/bin/install -c -m 644 ./pldbgapi--1.0.sql ./pldbgapi--unpackaged--1.0.sql  '/opt/pgsql9.5.2/share/extension/'
	/usr/bin/install -c -m 644 ./README.pldebugger '/opt/pgsql9.5.2/share/doc//extension/'
	[root@localhost lib]# ll|grep debug
	-rwxr-xr-x. 1 root root  246413 May 19 00:20 plugin_debugger.so

4. create extension xxx; 对于一些老版本的插件, 可能没有改成扩展插件的安装模式. 那么需要到对应的库中直接执行SQL.
	
	[root@localhost lib]# su - postgres
	Last login: Wed May 18 16:59:05 CST 2016 on pts/0
	postgres@localhost-> psql
	psql (9.5.2)
	Type "help" for help.
	
	postgres=# create extension pldbgpi;
	ERROR:  could not open extension control file "/opt/pgsql9.5.2/share/extension/pldbgpi.control": No such file or directory
	postgres=# create extension pldbgapi;
	CREATE EXTENSION
	postgres=# \df ----如图3 
	
![Alt text](/assets/blog-images/20160519102355.png)

以上步骤，就已经把安装包安装成功，下面就是验证

4. 需要安装pgadmin，具体步骤

	1） 打开网页http://www.pgadmin.org/，点击标签页download，如图4：

![Alt text](/assets/blog-images/20160519102839.png)
	
	2） 点击进入，如图5：

![Alt text](/assets/blog-images/20160519103020.png)

	3）点击进入，如图6：

![Alt text](/assets/blog-images/20160519103149.png)

	4）解压，点击pgadmin3.msi进行安装，一路next即可

5. 配置连接postgres，如图7：

![Alt text](/assets/blog-images/20160519103448.jpg)

6. 在postgres库，新创建一个函数f_test()
	
	postgres@localhost-> psql
	psql (9.5.2)
	Type "help" for help.
	
	postgres=# create function f_test(i_id int) returns int as $$
	postgres$# declare 
	postgres$# begin
	postgres$#   return i_id*2;
	postgres$# end;
	postgres$# $$ language plpgsql strict;
	CREATE FUNCTION
	
	然后在pgadmin中，如图8：

![Alt text](/assets/blog-images/20160519104033.png)

进入调试窗口，上图9：

![Alt text](/assets/blog-images/20160519104329.jpg)

这里说一下，在步骤5中，pgadmin配置连接postgres，一定要注意防火墙，以及端口的放开访问，要不连不上了。
ok！