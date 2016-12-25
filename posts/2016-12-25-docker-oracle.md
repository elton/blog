---
layout: post
title: 'Mac下使用Docker制作Oralce 12c镜像'
description: '使用Docker容器来制作安装和配置Oracle 12c，并安装示例数据库'
date: 2016-12-25
comments: true
categories:
- Mac
tags:
- docker 
- Oracle

---

#Mac下使用Docker制作Oralce 12c镜像

##安装Docker

在[官网](https://docs.docker.com/docker-for-mac/)下载最新版的Docker for Mac，安装好以后，使用下面命令测试
	
```
$ docker --version
Docker version 1.12.5, build 7392c3b	

$ docker run -d -p 80:80 --name webserver nginx
```
在浏览器中使用<http://localhost/>查看，如果安装正确，将显示如下界面。
![nginx](https://docs.docker.com/docker-for-mac/images/hello-world-nginx.png)


##制作镜像

###下载制作镜像要用的代码  

```
$git clone https://github.com/wscherphof/oracle-12c
```

###下载Oracle数据库

####第一步
1. 从 [Oracle Tech Net](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/database12c-linux-download-2240591.html) 下载 linuxamd64\_12102\_database\_1of2.zip & linuxamd64\_12102\_database\_2of2.zip
2. 把两个zip文件放到```step1```目录中
3. 进入 cd ```oracle-12c```目录
4. ```$ docker build -t oracle-12c:step1 step1```
5. ```$ docker run --shm-size=4g -ti --name step1 oracle-12c:step1 /bin/bash```
6. ```# /tmp/install/install``` (大约5分钟)  

	```
	Tue Sep 16 08:48:00 UTC 2014
	Starting Oracle Universal Installer...
	
	Checking Temp space: must be greater than 500 MB.   Actual 40142 MB    Passed
	Checking swap space: must be greater than 150 MB.   Actual 1392 MB    Passed
	Preparing to launch Oracle Universal Installer from /tmp/OraInstall2014-09-16_08-48-01AM. Please wait ...[root@51905aa48207 /]# You can find the log of this install session at:
	 /u01/app/oraInventory/logs/installActions2014-09-16_08-48-01AM.log
	The installation of Oracle Database 12c was successful.
	Please check '/u01/app/oraInventory/logs/silentInstall2014-09-16_08-48-01AM.log' for more details.
	
	As a root user, execute the following script(s):
		1. /u01/app/oracle/product/12.1.0/dbhome_1/root.sh
	
	
	
	Successfully Setup Software.
	As install user, execute the following script to complete the configuration.
		1. /u01/app/oracle/product/12.1.0/dbhome_1/cfgtoollogs/configToolAllCommands RESPONSE_FILE=<response_file>
	
	 	Note:
		1. This script must be run on the same host from where installer was run. 
		2. This script needs a small password properties file for configuration assistants that require passwords (refer to install guide documentation).
	
	```
7. ` <enter>`
8. ` # exit` 
9. `$ docker commit step1 oracle-12c:installed`

####第二步
1. `$ docker build -t oracle-12c:step2 step2`
2. `$ docker run --shm-size=4g -ti --name step2 oracle-12c:step2 /bin/bash`
3. ` # /tmp/create` （大约需要15分钟）

	```
	Tue Sep 16 11:07:30 UTC 2014
	Creating database...
	
	SQL*Plus: Release 12.1.0.2.0 Production on Tue Sep 16 11:07:30 2014
	
	Copyright (c) 1982, 2014, Oracle.  All rights reserved.
	
	Connected to an idle instance.
	
	File created.
	
	ORACLE instance started.
	
	Total System Global Area 1073741824 bytes
	Fixed Size		    2932632 bytes
	Variable Size		  721420392 bytes
	Database Buffers	  343932928 bytes
	Redo Buffers		    5455872 bytes
	
	Database created.
	
	
	Tablespace created.
	
	
	Tablespace created.
	
	Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
	With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
	
	Tue Sep 16 11:07:50 UTC 2014
	Creating password file...
	
	Tue Sep 16 11:07:50 UTC 2014
	Running catalog.sql...
	
	Tue Sep 16 11:08:51 UTC 2014
	Running catproc.sql...
	
	Tue Sep 16 11:19:38 UTC 2014
	Running pupbld.sql...
	
	Tue Sep 16 11:19:38 UTC 2014
	Create is done; commit the container now
	```
4. ` # exit`
5. `$ docker commit step2 oracle-12c:created`

####第三步
1. `$ docker build -t oracle-12c step3`  
之后就生成了一个名为oracle-12c的docker image了，这时候可以把其他的中间使用的image都删掉了。

```
$ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
oracle-12c                  latest              17c7e6958ec3        12 hours ago        14.73 GB
wscherphof/oracle-linux-7   latest              a07f8cc9627f        2 years ago         437.4 MB
```

###运行

```
$ docker run --shm-size=4g -dP --name orcl oracle-12c
989f1b41b1f00c53576ab85e773b60f2458a75c108c12d4ac3d70be4e801b563
```

* 默认密码 
 
username | password
-------- | -------------------
sys      | change\_on\_install
system   | manager

```ORCL``` 数据库端口是 ```1521```, 通过下面的命令，可以查找主机的端口

```
$ docker port orcl 1521
0.0.0.0:32770
```

这样就可以使用 ```sqlplus``` [^1] 的命令连接Oracle 

[^1]: 安装 **sqlplus**。从Oracle网站下载Mac版本的 [客户端](http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html)。下载```instantclient-basic-macos.x64-12.1.0.2.0.zip```和```instantclient-sqlplus-macos.x64-12.1.0.2.0.zip```，把它们都解压到如```instantclient_12_1```的目录中。进入```cd ~/instantclient_12_1```目录， ```ln -s libclntsh.dylib.12.1 libclntsh.dylib```，```ln -s libocci.dylib.12.1 libocci.dylib```，设置环境变量 ```export PATH=~/instantclient_12_1:$PATH```

```
$ sqlplus system/manager@localhost:32770/orcl

SQL*Plus: Release 12.1.0.2.0 Production on Sun Dec 25 00:32:23 2016

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Last Successful login time: Sat Dec 24 2016 23:25:27 +08:00

Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL> 
```

###进入容器
如果想进入容器进行操作，可以使用下面的命令：

```
docker exec -it orcl /bin/bash
```

###安装Sample数据库
默认情况下，sample数据库和对应的用户都没有创建。

```
# 进入容器
$ docker exec -it orcl /bin/bash

#安装git
$ yum install git

#进入demo schema目录
$ cd /u01/app/oracle/product/12.1.0/dbhome_1/demo/schema/

#下载Sample schema
$ git clone https://github.com/oracle/db-sample-schemas.git .

#替换下载脚本中的目录
$ perl -p -i.bak -e 's#__SUB__CWD__#'$(pwd)'#g' *.sql */*.sql */*.dat

#创建log目录
$ mkdir log

#更改权限
$ chown -R oracle:oinstall .

#连接数据库
$ sqlplus sys/change_on_install@localhost/orcl as sysdb

#执行安装脚本
#格式为SQL> @?/demo/schema/mksample <SYSTEM_password> <SYS_password>
 		<HR_password> <OE_password> <PM_password> <IX_password> 
		<SH_password> <BI_password> EXAMPLE TEMP 
		$ORACLE_HOME/demo/schema/log/ localhost:1521/pdb
		
SQL> @?/demo/schema/mksample manager change_on_install hr oe pm ix sh bi users temp $ORACLE_HOME/demo/schema/log/ localhost:1521/orcl

#最后可以看到执行结果
All named objects and stati

OWNER  OBJECT_TYPE          OBJECT_NAME                    SUBOBJECT_NAME   STATUS
------ -------------------- ------------------------------ ---------------- --------
BI     SYNONYM              CHANNELS                                        VALID
BI     SYNONYM              COSTS                                           VALID
BI     SYNONYM              COUNTRIES                                       VALID
BI     SYNONYM              CUSTOMERS                                       VALID
BI     SYNONYM              PRODUCTS                                        VALID
BI     SYNONYM              PROMOTIONS                                      VALID
BI     SYNONYM              SALES                                           VALID
BI     SYNONYM              TIMES                                           VALID
HR     INDEX                COUNTRY_C_ID_PK                                 VALID
HR     INDEX                DEPT_ID_PK                                      VALID
HR     INDEX                DEPT_LOCATION_IX                                VALID
HR     INDEX                EMP_DEPARTMENT_IX                               VALID
HR     INDEX                EMP_EMAIL_UK                                    VALID
HR     INDEX                EMP_EMP_ID_PK                                   VALID
HR     INDEX                EMP_JOB_IX                                      VALID
HR     INDEX                EMP_MANAGER_IX                                  VALID
HR     INDEX                EMP_NAME_IX                                     VALID
HR     INDEX                JHIST_DEPARTMENT_IX                             VALID
HR     INDEX                JHIST_EMPLOYEE_IX                               VALID
HR     INDEX                JHIST_EMP_ID_ST_DATE_PK                         VALID
HR     INDEX                JHIST_JOB_IX                                    VALID
HR     INDEX                JOB_ID_PK                                       VALID
HR     INDEX                LOC_CITY_IX                                     VALID
HR     INDEX                LOC_COUNTRY_IX                                  VALID
HR     INDEX                LOC_ID_PK                                       VALID
HR     INDEX                LOC_STATE_PROVINCE_IX                           VALID
HR     INDEX                REG_ID_PK                                       VALID
HR     PROCEDURE            ADD_JOB_HISTORY                                 VALID
HR     PROCEDURE            SECURE_DML                                      VALID
HR     SEQUENCE             DEPARTMENTS_SEQ                                 VALID
HR     SEQUENCE             EMPLOYEES_SEQ                                   VALID
HR     SEQUENCE             LOCATIONS_SEQ                                   VALID
HR     TABLE                COUNTRIES                                       VALID
HR     TABLE                DEPARTMENTS                                     VALID
HR     TABLE                EMPLOYEES                                       VALID
HR     TABLE                JOBS                                            VALID
HR     TABLE                JOB_HISTORY                                     VALID
HR     TABLE                LOCATIONS                                       VALID
HR     TABLE                REGIONS                                         VALID
HR     TRIGGER              SECURE_EMPLOYEES                                VALID
HR     TRIGGER              UPDATE_JOB_HISTORY                              VALID
HR     VIEW                 EMP_DETAILS_VIEW                                VALID
IX     EVALUATION CONTEXT   AQ$_ORDERS_QUEUETABLE_V                         VALID
IX     EVALUATION CONTEXT   AQ$_STREAMS_QUEUE_TABLE_V                       VALID
IX     INDEX                AQ$_STREAMS_QUEUE_TABLE_Y                       VALID
IX     QUEUE                AQ$_ORDERS_QUEUETABLE_E                         VALID
IX     QUEUE                AQ$_STREAMS_QUEUE_TABLE_E                       VALID
IX     QUEUE                ORDERS_QUEUE                                    VALID
IX     QUEUE                STREAMS_QUEUE                                   VALID
IX     RULE SET             ORDERS_QUEUE_N                                  VALID
IX     RULE SET             ORDERS_QUEUE_R                                  VALID
IX     RULE SET             STREAMS_QUEUE_N                                 VALID
IX     RULE SET             STREAMS_QUEUE_R                                 VALID
IX     SEQUENCE             AQ$_ORDERS_QUEUETABLE_N                         VALID
IX     SEQUENCE             AQ$_STREAMS_QUEUE_TABLE_N                       VALID
IX     TABLE                AQ$_ORDERS_QUEUETABLE_G                         VALID
IX     TABLE                AQ$_ORDERS_QUEUETABLE_H                         VALID
IX     TABLE                AQ$_ORDERS_QUEUETABLE_I                         VALID
IX     TABLE                AQ$_ORDERS_QUEUETABLE_L                         VALID
IX     TABLE                AQ$_ORDERS_QUEUETABLE_S                         VALID
IX     TABLE                AQ$_ORDERS_QUEUETABLE_T                         VALID
IX     TABLE                AQ$_STREAMS_QUEUE_TABLE_C                       VALID
IX     TABLE                AQ$_STREAMS_QUEUE_TABLE_G                       VALID
IX     TABLE                AQ$_STREAMS_QUEUE_TABLE_H                       VALID
IX     TABLE                AQ$_STREAMS_QUEUE_TABLE_I                       VALID
IX     TABLE                AQ$_STREAMS_QUEUE_TABLE_L                       VALID
IX     TABLE                AQ$_STREAMS_QUEUE_TABLE_S                       VALID
IX     TABLE                AQ$_STREAMS_QUEUE_TABLE_T                       VALID
IX     TABLE                ORDERS_QUEUETABLE                               VALID
IX     TABLE                STREAMS_QUEUE_TABLE                             VALID
IX     TYPE                 ORDER_EVENT_TYP                                 VALID
IX     VIEW                 AQ$ORDERS_QUEUETABLE                            VALID
IX     VIEW                 AQ$ORDERS_QUEUETABLE_R                          VALID
IX     VIEW                 AQ$ORDERS_QUEUETABLE_S                          VALID
IX     VIEW                 AQ$STREAMS_QUEUE_TABLE                          VALID
IX     VIEW                 AQ$STREAMS_QUEUE_TABLE_R                        VALID
IX     VIEW                 AQ$STREAMS_QUEUE_TABLE_S                        VALID
IX     VIEW                 AQ$_ORDERS_QUEUETABLE_F                         VALID
IX     VIEW                 AQ$_STREAMS_QUEUE_TABLE_F                       VALID
OE     FUNCTION             GET_PHONE_NUMBER_F                              VALID
OE     INDEX                ACTION_TABLE_MEMBERS                            VALID
OE     INDEX                INVENTORY_IX                                    VALID
OE     INDEX                INV_PRODUCT_IX                                  VALID
OE     INDEX                ITEM_ORDER_IX                                   VALID
OE     INDEX                ITEM_PRODUCT_IX                                 VALID
OE     INDEX                LINEITEM_TABLE_MEMBERS                          VALID
OE     INDEX                ORDER_ITEMS_PK                                  VALID
OE     INDEX                ORDER_ITEMS_UK                                  VALID
OE     INDEX                ORDER_PK                                        VALID
OE     INDEX                ORD_CUSTOMER_IX                                 VALID
OE     INDEX                ORD_ORDER_DATE_IX                               VALID
OE     INDEX                ORD_SALES_REP_IX                                VALID
OE     INDEX                PRD_DESC_PK                                     VALID
OE     INDEX                PRODUCT_INFORMATION_PK                          VALID
OE     INDEX                PROD_NAME_IX                                    VALID
OE     INDEX                PROD_SUPPLIER_IX                                VALID
OE     INDEX                PROMO_ID_PK                                     VALID
OE     LOB                  EXTRADATA206_L                                  VALID
OE     LOB                  NAMESPACES207_L                                 VALID
OE     SEQUENCE             ORDERS_SEQ                                      VALID
OE     SYNONYM              COUNTRIES                                       VALID
OE     SYNONYM              DEPARTMENTS                                     VALID
OE     SYNONYM              EMPLOYEES                                       VALID
OE     SYNONYM              JOBS                                            VALID
OE     SYNONYM              JOB_HISTORY                                     VALID
OE     SYNONYM              LOCATIONS                                       VALID
OE     TABLE                ACTION_TABLE                                    VALID
OE     TABLE                CATEGORIES_TAB                                  VALID
OE     TABLE                INVENTORIES                                     VALID
OE     TABLE                LINEITEM_TABLE                                  VALID
OE     TABLE                ORDERS                                          VALID
OE     TABLE                ORDER_ITEMS                                     VALID
OE     TABLE                PRODUCT_DESCRIPTIONS                            VALID
OE     TABLE                PRODUCT_INFORMATION                             VALID
OE     TABLE                PRODUCT_REF_LIST_NESTEDTAB                      VALID
OE     TABLE                PROMOTIONS                                      VALID
OE     TABLE                PURCHASEORDER                                   VALID
OE     TABLE                SUBCATEGORY_REF_LIST_NESTEDTAB                  VALID
OE     TRIGGER              INSERT_ORD_LINE                                 VALID
OE     TRIGGER              PURCHASEORDER$xd                                VALID
OE     TYPE                 ACTIONS_T                                       VALID
OE     TYPE                 ACTION_T                                        VALID
OE     TYPE                 ACTION_V                                        VALID
OE     TYPE                 CATALOG_TYP                    $VSN_1           VALID
OE     TYPE                 CATALOG_TYP                                     VALID
OE     TYPE                 CATEGORY_TYP                   $VSN_1           VALID
OE     TYPE                 CATEGORY_TYP                                    VALID
OE     TYPE                 COMPOSITE_CATEGORY_TYP         $VSN_1           VALID
OE     TYPE                 COMPOSITE_CATEGORY_TYP                          VALID
OE     TYPE                 CORPORATE_CUSTOMER_TYP                          VALID
OE     TYPE                 CUSTOMER_TYP                                    VALID
OE     TYPE                 CUST_ADDRESS_TYP                                VALID
OE     TYPE                 INVENTORY_LIST_TYP                              VALID
OE     TYPE                 INVENTORY_TYP                                   VALID
OE     TYPE                 LEAF_CATEGORY_TYP              $VSN_1           VALID
OE     TYPE                 LEAF_CATEGORY_TYP                               VALID
OE     TYPE                 LINEITEMS_T                                     VALID
OE     TYPE                 LINEITEM_T                                      VALID
OE     TYPE                 LINEITEM_V                                      VALID
OE     TYPE                 ORDER_ITEM_LIST_TYP                             VALID
OE     TYPE                 ORDER_ITEM_TYP                                  VALID
OE     TYPE                 ORDER_LIST_TYP                                  VALID
OE     TYPE                 ORDER_TYP                                       VALID
OE     TYPE                 PART_T                                          VALID
OE     TYPE                 PHONE_LIST_TYP                                  VALID
OE     TYPE                 PRODUCT_INFORMATION_TYP                         VALID
OE     TYPE                 PRODUCT_REF_LIST_TYP                            VALID
OE     TYPE                 PURCHASEORDER_T                                 VALID
OE     TYPE                 REJECTION_T                                     VALID
OE     TYPE                 SHIPPING_INSTRUCTIONS_T                         VALID
OE     TYPE                 SUBCATEGORY_REF_LIST_TYP                        VALID
OE     TYPE                 WAREHOUSE_TYP                                   VALID
OE     TYPE BODY            CATALOG_TYP                                     VALID
OE     TYPE BODY            COMPOSITE_CATEGORY_TYP                          VALID
OE     TYPE BODY            LEAF_CATEGORY_TYP                               VALID
OE     VIEW                 ORDERS_VIEW                                     VALID
OE     VIEW                 PRODUCTS                                        VALID
OE     VIEW                 PRODUCT_PRICES                                  VALID
PM     INDEX                PRINTMEDIA_PK                                   VALID
PM     TABLE                PRINT_MEDIA                                     VALID
PM     TABLE                TEXTDOCS_NESTEDTAB                              VALID
PM     TYPE                 ADHEADER_TYP                                    VALID
PM     TYPE                 TEXTDOC_TAB                                     VALID
PM     TYPE                 TEXTDOC_TYP                                     VALID
SH     DIMENSION            CHANNELS_DIM                                    VALID
SH     DIMENSION            CUSTOMERS_DIM                                   VALID
SH     DIMENSION            PRODUCTS_DIM                                    VALID
SH     DIMENSION            PROMOTIONS_DIM                                  VALID
SH     DIMENSION            TIMES_DIM                                       VALID
SH     INDEX                CHANNELS_PK                                     VALID
SH     INDEX                COSTS_PROD_BIX                                  VALID
SH     INDEX                COSTS_TIME_BIX                                  VALID
SH     INDEX                COUNTRIES_PK                                    VALID
SH     INDEX                CUSTOMERS_GENDER_BIX                            VALID
SH     INDEX                CUSTOMERS_MARITAL_BIX                           VALID
SH     INDEX                CUSTOMERS_PK                                    VALID
SH     INDEX                CUSTOMERS_YOB_BIX                               VALID
SH     INDEX                FW_PSC_S_MV_CHAN_BIX                            VALID
SH     INDEX                FW_PSC_S_MV_PROMO_BIX                           VALID
SH     INDEX                FW_PSC_S_MV_SUBCAT_BIX                          VALID
SH     INDEX                FW_PSC_S_MV_WD_BIX                              VALID
SH     INDEX                PRODUCTS_PK                                     VALID
SH     INDEX                PRODUCTS_PROD_CAT_IX                            VALID
SH     INDEX                PRODUCTS_PROD_STATUS_BIX                        VALID
SH     INDEX                PRODUCTS_PROD_SUBCAT_IX                         VALID
SH     INDEX                PROMO_PK                                        VALID
SH     INDEX                SALES_CHANNEL_BIX                               VALID
SH     INDEX                SALES_CUST_BIX                                  VALID
SH     INDEX                SALES_PROD_BIX                                  VALID
SH     INDEX                SALES_PROMO_BIX                                 VALID
SH     INDEX                SALES_TIME_BIX                                  VALID
SH     INDEX                TIMES_PK                                        VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_1995       VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_1996       VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_H1_1997    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_H2_1997    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q1_1998    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q1_1999    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q1_2000    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q1_2001    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q1_2002    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q1_2003    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q2_1998    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q2_1999    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q2_2000    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q2_2001    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q2_2002    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q2_2003    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q3_1998    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q3_1999    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q3_2000    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q3_2001    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q3_2002    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q3_2003    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q4_1998    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q4_1999    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q4_2000    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q4_2001    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q4_2002    VALID
SH     INDEX PARTITION      COSTS_PROD_BIX                 COSTS_Q4_2003    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_1995       VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_1996       VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_H1_1997    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_H2_1997    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q1_1998    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q1_1999    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q1_2000    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q1_2001    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q1_2002    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q1_2003    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q2_1998    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q2_1999    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q2_2000    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q2_2001    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q2_2002    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q2_2003    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q3_1998    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q3_1999    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q3_2000    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q3_2001    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q3_2002    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q3_2003    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q4_1998    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q4_1999    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q4_2000    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q4_2001    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q4_2002    VALID
SH     INDEX PARTITION      COSTS_TIME_BIX                 COSTS_Q4_2003    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_1995       VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_1996       VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_H1_1997    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_H2_1997    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q1_1998    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q1_1999    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q1_2000    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q1_2001    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q1_2002    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q1_2003    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q2_1998    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q2_1999    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q2_2000    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q2_2001    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q2_2002    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q2_2003    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q3_1998    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q3_1999    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q3_2000    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q3_2001    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q3_2002    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q3_2003    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q4_1998    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q4_1999    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q4_2000    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q4_2001    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q4_2002    VALID
SH     INDEX PARTITION      SALES_CHANNEL_BIX              SALES_Q4_2003    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_1995       VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_1996       VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_H1_1997    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_H2_1997    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q1_1998    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q1_1999    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q1_2000    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q1_2001    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q1_2002    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q1_2003    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q2_1998    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q2_1999    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q2_2000    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q2_2001    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q2_2002    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q2_2003    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q3_1998    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q3_1999    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q3_2000    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q3_2001    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q3_2002    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q3_2003    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q4_1998    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q4_1999    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q4_2000    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q4_2001    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q4_2002    VALID
SH     INDEX PARTITION      SALES_CUST_BIX                 SALES_Q4_2003    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_1995       VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_1996       VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_H1_1997    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_H2_1997    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q1_1998    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q1_1999    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q1_2000    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q1_2001    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q1_2002    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q1_2003    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q2_1998    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q2_1999    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q2_2000    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q2_2001    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q2_2002    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q2_2003    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q3_1998    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q3_1999    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q3_2000    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q3_2001    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q3_2002    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q3_2003    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q4_1998    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q4_1999    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q4_2000    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q4_2001    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q4_2002    VALID
SH     INDEX PARTITION      SALES_PROD_BIX                 SALES_Q4_2003    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_1995       VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_1996       VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_H1_1997    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_H2_1997    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q1_1998    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q1_1999    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q1_2000    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q1_2001    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q1_2002    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q1_2003    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q2_1998    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q2_1999    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q2_2000    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q2_2001    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q2_2002    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q2_2003    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q3_1998    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q3_1999    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q3_2000    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q3_2001    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q3_2002    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q3_2003    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q4_1998    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q4_1999    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q4_2000    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q4_2001    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q4_2002    VALID
SH     INDEX PARTITION      SALES_PROMO_BIX                SALES_Q4_2003    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_1995       VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_1996       VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_H1_1997    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_H2_1997    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q1_1998    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q1_1999    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q1_2000    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q1_2001    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q1_2002    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q1_2003    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q2_1998    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q2_1999    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q2_2000    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q2_2001    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q2_2002    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q2_2003    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q3_1998    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q3_1999    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q3_2000    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q3_2001    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q3_2002    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q3_2003    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q4_1998    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q4_1999    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q4_2000    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q4_2001    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q4_2002    VALID
SH     INDEX PARTITION      SALES_TIME_BIX                 SALES_Q4_2003    VALID
SH     MATERIALIZED VIEW    CAL_MONTH_SALES_MV                              VALID
SH     MATERIALIZED VIEW    FWEEK_PSCAT_SALES_MV                            VALID
SH     TABLE                CAL_MONTH_SALES_MV                              VALID
SH     TABLE                CHANNELS                                        VALID
SH     TABLE                COSTS                                           VALID
SH     TABLE                COUNTRIES                                       VALID
SH     TABLE                CUSTOMERS                                       VALID
SH     TABLE                FWEEK_PSCAT_SALES_MV                            VALID
SH     TABLE                PRODUCTS                                        VALID
SH     TABLE                PROMOTIONS                                      VALID
SH     TABLE                SALES                                           VALID
SH     TABLE                SALES_TRANSACTIONS_EXT                          VALID
SH     TABLE                SUPPLEMENTARY_DEMOGRAPHICS                      VALID
SH     TABLE                TIMES                                           VALID
SH     TABLE PARTITION      COSTS                          COSTS_1995       VALID
SH     TABLE PARTITION      COSTS                          COSTS_1996       VALID
SH     TABLE PARTITION      COSTS                          COSTS_H1_1997    VALID
SH     TABLE PARTITION      COSTS                          COSTS_H2_1997    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q1_1998    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q1_1999    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q1_2000    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q1_2001    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q1_2002    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q1_2003    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q2_1998    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q2_1999    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q2_2000    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q2_2001    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q2_2002    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q2_2003    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q3_1998    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q3_1999    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q3_2000    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q3_2001    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q3_2002    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q3_2003    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q4_1998    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q4_1999    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q4_2000    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q4_2001    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q4_2002    VALID
SH     TABLE PARTITION      COSTS                          COSTS_Q4_2003    VALID
SH     TABLE PARTITION      SALES                          SALES_1995       VALID
SH     TABLE PARTITION      SALES                          SALES_1996       VALID
SH     TABLE PARTITION      SALES                          SALES_H1_1997    VALID
SH     TABLE PARTITION      SALES                          SALES_H2_1997    VALID
SH     TABLE PARTITION      SALES                          SALES_Q1_1998    VALID
SH     TABLE PARTITION      SALES                          SALES_Q1_1999    VALID
SH     TABLE PARTITION      SALES                          SALES_Q1_2000    VALID
SH     TABLE PARTITION      SALES                          SALES_Q1_2001    VALID
SH     TABLE PARTITION      SALES                          SALES_Q1_2002    VALID
SH     TABLE PARTITION      SALES                          SALES_Q1_2003    VALID
SH     TABLE PARTITION      SALES                          SALES_Q2_1998    VALID
SH     TABLE PARTITION      SALES                          SALES_Q2_1999    VALID
SH     TABLE PARTITION      SALES                          SALES_Q2_2000    VALID
SH     TABLE PARTITION      SALES                          SALES_Q2_2001    VALID
SH     TABLE PARTITION      SALES                          SALES_Q2_2002    VALID
SH     TABLE PARTITION      SALES                          SALES_Q2_2003    VALID
SH     TABLE PARTITION      SALES                          SALES_Q3_1998    VALID
SH     TABLE PARTITION      SALES                          SALES_Q3_1999    VALID
SH     TABLE PARTITION      SALES                          SALES_Q3_2000    VALID
SH     TABLE PARTITION      SALES                          SALES_Q3_2001    VALID
SH     TABLE PARTITION      SALES                          SALES_Q3_2002    VALID
SH     TABLE PARTITION      SALES                          SALES_Q3_2003    VALID
SH     TABLE PARTITION      SALES                          SALES_Q4_1998    VALID
SH     TABLE PARTITION      SALES                          SALES_Q4_1999    VALID
SH     TABLE PARTITION      SALES                          SALES_Q4_2000    VALID
SH     TABLE PARTITION      SALES                          SALES_Q4_2001    VALID
SH     TABLE PARTITION      SALES                          SALES_Q4_2002    VALID
SH     TABLE PARTITION      SALES                          SALES_Q4_2003    VALID
SH     VIEW                 PROFITS                                         VALID

459 rows selected.


Data types used

OWNER  DATA_TYPE                           DATA_TYPE_OWNER  DAT   COUNT(*)
------ ----------------------------------- ---------------- --- ----------
PM     ADHEADER_TYP                        PM                            1
IX     ANYDATA                             SYS                           7
IX     AQ$_SIG_PROP                        SYS                           4
PM     BFILE                                                             1
PM     BLOB                                                              2
HR     CHAR                                                              3
IX     CHAR                                                              2
SH     CHAR                                                              4
IX     CLOB                                                              2
PM     CLOB                                                              2
HR     DATE                                                              3
IX     DATE                                                              6
OE     DATE                                                              1
SH     DATE                                                             19
OE     INTERVAL YEAR(2) TO MONTH                                         2
PM     NCLOB                                                             1
HR     NUMBER                                                           21
IX     NUMBER                                                          107
OE     NUMBER                                                           40
PM     NUMBER                                                            2
SH     NUMBER                                                           94
OE     NVARCHAR2                                                         4
IX     ORDER_EVENT_TYP                     IX                            3
IX     RAW                                                              32
IX     ROWID                                                             6
PM     TEXTDOC_TAB                         PM                            1
IX     TIMESTAMP(6)                                                     25
OE     TIMESTAMP(6) WITH LOCAL TIME ZONE                                 1
IX     TIMESTAMP(6) WITH TIME ZONE                                       8
IX     TIMESTAMP(9)                                                      2
HR     VARCHAR2                                                         24
IX     VARCHAR2                                                        128
OE     VARCHAR2                                                         13
SH     VARCHAR2                                                         46
OE     XMLTYPE                             SYS                           1

35 rows selected.


XML tables

OWNER  TABLE_NAME                     SCHEMA_OWNER     STORAGE_TYPE
------ ------------------------------ ---------------- --------------------
OE     PURCHASEORDER                  OE               OBJECT-RELATIONAL

1 row selected.


All objects named 'SYS%' (LOBs etc)

OWNER  OBJECT_TYPE          STATUS     COUNT(*)
------ -------------------- -------- ----------
IX     INDEX                VALID            16
OE     INDEX                VALID            19
PM     INDEX                VALID             9
IX     LOB                  VALID             3
OE     LOB                  VALID             8
PM     LOB                  VALID             7
IX     TABLE                VALID             2

7 rows selected.


All constraints

OWNER  CONSTRAINT_TYPE      STATUS   VALIDATED        GENERATED          COUNT(*)
------ -------------------- -------- ---------------- ---------------- ----------
IX     Check or Not Null    ENABLED  VALIDATED        GENERATED NAME            4
OE     Check or Not Null    ENABLED  VALIDATED        GENERATED NAME            2
SH     Check or Not Null    ENABLED  VALIDATED        GENERATED NAME          115
HR     Check or Not Null    ENABLED  VALIDATED        USER NAME                15
OE     Check or Not Null    ENABLED  VALIDATED        USER NAME                 9
SH     Foreign key          DISABLED NOT VALIDATED    USER NAME                 2
OE     Foreign key          ENABLED  NOT VALIDATED    USER NAME                 1
SH     Foreign key          ENABLED  NOT VALIDATED    USER NAME                 8
HR     Foreign key          ENABLED  VALIDATED        USER NAME                10
OE     Foreign key          ENABLED  VALIDATED        USER NAME                 4
PM     Foreign key          ENABLED  VALIDATED        USER NAME                 1
SH     Primary key          DISABLED NOT VALIDATED    USER NAME                 1
SH     Primary key          ENABLED  NOT VALIDATED    USER NAME                 6
IX     Primary key          ENABLED  VALIDATED        GENERATED NAME           13
OE     Primary key          ENABLED  VALIDATED        GENERATED NAME            3
HR     Primary key          ENABLED  VALIDATED        USER NAME                 7
OE     Primary key          ENABLED  VALIDATED        USER NAME                 6
PM     Primary key          ENABLED  VALIDATED        USER NAME                 1
HR     Read only view       ENABLED  NOT VALIDATED    GENERATED NAME            1
IX     Read only view       ENABLED  NOT VALIDATED    GENERATED NAME            8
OE     Unique key           ENABLED  VALIDATED        GENERATED NAME            6
PM     Unique key           ENABLED  VALIDATED        GENERATED NAME            1
HR     Unique key           ENABLED  VALIDATED        USER NAME                 1

23 rows selected.


All dimensions

OWNER  DIMENSION_NAME       I COMPILE_STATE
------ -------------------- - -------------
SH     CHANNELS_DIM         N VALID
SH     CUSTOMERS_DIM        N VALID
SH     PRODUCTS_DIM         N VALID
SH     PROMOTIONS_DIM       N VALID
SH     TIMES_DIM            N VALID

5 rows selected.


All granted roles

GRANTED_ROLE              GRANTEE
------------------------- -------
AQ_ADMINISTRATOR_ROLE     IX
AQ_USER_ROLE              IX
CONNECT                   IX
CONNECT                   PM
RESOURCE                  BI
RESOURCE                  HR
RESOURCE                  IX
RESOURCE                  OE
RESOURCE                  PM
RESOURCE                  SH
SELECT_CATALOG_ROLE       IX
SELECT_CATALOG_ROLE       SH
XDBADMIN                  OE

13 rows selected.


All granted system privileges

PRIVILEGE                 GRANTEE
------------------------- -------
ALTER SESSION             BI
ALTER SESSION             HR
ALTER SESSION             IX
ALTER SESSION             SH
CREATE CLUSTER            BI
CREATE CLUSTER            IX
CREATE CLUSTER            SH
CREATE DATABASE LINK      BI
CREATE DATABASE LINK      HR
CREATE DATABASE LINK      IX
CREATE DATABASE LINK      OE
CREATE DATABASE LINK      SH
CREATE DIMENSION          SH
CREATE INDEXTYPE          IX
CREATE MATERIALIZED VIEW  OE
CREATE MATERIALIZED VIEW  SH
CREATE OPERATOR           IX
CREATE PROCEDURE          IX
CREATE RULE               IX
CREATE RULE SET           IX
CREATE SEQUENCE           BI
CREATE SEQUENCE           HR
CREATE SEQUENCE           IX
CREATE SEQUENCE           SH
CREATE SESSION            BI
CREATE SESSION            HR
CREATE SESSION            IX
CREATE SESSION            OE
CREATE SESSION            SH
CREATE SYNONYM            BI
CREATE SYNONYM            HR
CREATE SYNONYM            IX
CREATE SYNONYM            OE
CREATE SYNONYM            SH
CREATE TABLE              BI
CREATE TABLE              IX
CREATE TABLE              SH
CREATE TRIGGER            IX
CREATE TYPE               IX
CREATE VIEW               BI
CREATE VIEW               HR
CREATE VIEW               IX
CREATE VIEW               OE
CREATE VIEW               SH
QUERY REWRITE             OE
QUERY REWRITE             SH
SELECT ANY DICTIONARY     IX
UNLIMITED TABLESPACE      BI
UNLIMITED TABLESPACE      HR
UNLIMITED TABLESPACE      IX
UNLIMITED TABLESPACE      OE
UNLIMITED TABLESPACE      PM
UNLIMITED TABLESPACE      SH

53 rows selected.


All granted object privileges

OWNER  TABLE_NAME                     PRIVILEGE                 GRANTEE
------ ------------------------------ ------------------------- -------
HR     COUNTRIES                      REFERENCES                OE
HR     COUNTRIES                      SELECT                    OE
HR     DEPARTMENTS                    SELECT                    OE
HR     EMPLOYEES                      REFERENCES                OE
HR     EMPLOYEES                      SELECT                    OE
HR     JOBS                           SELECT                    OE
HR     JOB_HISTORY                    SELECT                    OE
HR     LOCATIONS                      REFERENCES                OE
HR     LOCATIONS                      SELECT                    OE
OE     INVENTORIES                    SELECT                    BI
OE     INVENTORIES                    SELECT                    PM
OE     ORDERS                         SELECT                    BI
OE     ORDERS                         SELECT                    PM
OE     ORDER_ITEMS                    SELECT                    BI
OE     ORDER_ITEMS                    SELECT                    PM
OE     PRODUCTS                       SELECT                    BI
OE     PRODUCT_DESCRIPTIONS           SELECT                    BI
OE     PRODUCT_DESCRIPTIONS           SELECT                    PM
OE     PRODUCT_INFORMATION            REFERENCES                PM
OE     PRODUCT_INFORMATION            SELECT                    BI
OE     PRODUCT_INFORMATION            SELECT                    PM
OE     PRODUCT_PRICES                 SELECT                    BI
OE     PROMOTIONS                     SELECT                    BI
SH     CAL_MONTH_SALES_MV             SELECT                    BI
SH     CHANNELS                       SELECT                    BI
SH     COSTS                          SELECT                    BI
SH     COUNTRIES                      SELECT                    BI
SH     CUSTOMERS                      SELECT                    BI
SH     FWEEK_PSCAT_SALES_MV           SELECT                    BI
SH     PRODUCTS                       SELECT                    BI
SH     PROMOTIONS                     SELECT                    BI
SH     SALES                          SELECT                    BI
SH     TIMES                          SELECT                    BI
SYS    AQ$_UNFLUSHED_DEQUEUES         SELECT                    IX
SYS    DATA_FILE_DIR                  READ                      SH
SYS    DBMS_APPLY_ADM                 EXECUTE                   IX
SYS    DBMS_AQ                        EXECUTE                   IX
SYS    DBMS_AQADM                     EXECUTE                   IX
SYS    DBMS_AQ_BQVIEW                 EXECUTE                   IX
SYS    DBMS_CAPTURE_ADM               EXECUTE                   IX
SYS    DBMS_FLASHBACK                 EXECUTE                   IX
SYS    DBMS_PROPAGATION_ADM           EXECUTE                   IX
SYS    DBMS_STATS                     EXECUTE                   HR
SYS    DBMS_STATS                     EXECUTE                   IX
SYS    DBMS_STATS                     EXECUTE                   OE
SYS    DBMS_STATS                     EXECUTE                   PM
SYS    DBMS_STATS                     EXECUTE                   SH
SYS    DBMS_STREAMS_ADM               EXECUTE                   IX
SYS    LOG_FILE_DIR                   READ                      SH
SYS    LOG_FILE_DIR                   WRITE                     SH
SYS    MEDIA_DIR                      READ                      PM
SYS    QT19972_BUFFER                 SELECT                    IX
SYS    QT19999_BUFFER                 SELECT                    IX
SYS    SS_OE_XMLDIR                   READ                      OE
SYS    SS_OE_XMLDIR                   WRITE                     OE
SYS    SUBDIR                         READ                      OE
SYS    SUBDIR                         WRITE                     OE

57 rows selected.


Space usage

OWNER  SEGMENT_TYPE         SUM(BYTES)
------ -------------------- ----------
HR     INDEX                   1245184
HR     TABLE                    393216
HR                             1638400
IX     INDEX                    917504
IX     TABLE                    524288
IX     LOBINDEX                 196608
IX     LOBSEGMENT               393216
IX                             2031616
OE     INDEX                   2359296
OE     TABLE                   2686976
OE     LOBINDEX                 655360
OE     LOBSEGMENT              1310720
OE     NESTED TABLE             589824
OE                             7602176
PM     INDEX                    196608
PM     TABLE                     65536
PM     LOBINDEX                 458752
PM     LOBSEGMENT              5373952
PM     NESTED TABLE              65536
PM                             6160384
SH     INDEX                   2293760
SH     TABLE                  19726336
SH     INDEX PARTITION       680132608
SH     TABLE PARTITION       134217728
SH                           836370432
                             853803008

26 rows selected.


Table cardinality relational and object tables

OWNER  TABLE_NAME                       NUM_ROWS
------ ------------------------------ ----------
HR     COUNTRIES                              25
HR     DEPARTMENTS                            27
HR     EMPLOYEES                             107
HR     JOBS                                   19
HR     JOB_HISTORY                            10
HR     LOCATIONS                              23
HR     REGIONS                                 4
IX     AQ$_ORDERS_QUEUETABLE_G                 0
IX     AQ$_ORDERS_QUEUETABLE_H                 2
IX     AQ$_ORDERS_QUEUETABLE_I                 2
IX     AQ$_ORDERS_QUEUETABLE_L                 2
IX     AQ$_ORDERS_QUEUETABLE_S                 4
IX     AQ$_ORDERS_QUEUETABLE_T                 0
IX     AQ$_STREAMS_QUEUE_TABLE_C               0
IX     AQ$_STREAMS_QUEUE_TABLE_G               0
IX     AQ$_STREAMS_QUEUE_TABLE_H               0
IX     AQ$_STREAMS_QUEUE_TABLE_I               0
IX     AQ$_STREAMS_QUEUE_TABLE_L               0
IX     AQ$_STREAMS_QUEUE_TABLE_S               1
IX     AQ$_STREAMS_QUEUE_TABLE_T               0
IX     ORDERS_QUEUETABLE
IX     STREAMS_QUEUE_TABLE
IX     SYS_IOT_OVER_19986                      0
IX     SYS_IOT_OVER_20015                      0
OE     ACTION_TABLE                          132
OE     CATEGORIES_TAB                          4
OE     INVENTORIES                          1112
OE     LINEITEM_TABLE                       2232
OE     ORDERS                                105
OE     ORDER_ITEMS                           665
OE     PRODUCT_DESCRIPTIONS                 8640
OE     PRODUCT_INFORMATION                   288
OE     PRODUCT_REF_LIST_NESTEDTAB              0
OE     PROMOTIONS                              2
OE     PURCHASEORDER                         132
OE     SUBCATEGORY_REF_LIST_NESTEDTAB          3
PM     PRINT_MEDIA                             4
PM     TEXTDOCS_NESTEDTAB                     12
SH     CAL_MONTH_SALES_MV                     48
SH     CHANNELS                                5
SH     COSTS                                   0
SH     COUNTRIES                              23
SH     CUSTOMERS                           55500
SH     FWEEK_PSCAT_SALES_MV                11266
SH     PRODUCTS                               72
SH     PROMOTIONS                            503
SH     SALES                              918843
SH     SALES_TRANSACTIONS_EXT
SH     SUPPLEMENTARY_DEMOGRAPHICS           4500
SH     TIMES                                1826

50 rows selected.


Index cardinality (without  LOB indexes)

OWNER  INDEX_NAME                DISTINCT_KEYS   NUM_ROWS
------ ------------------------- ------------- ----------
HR     COUNTRY_C_ID_PK                      25         25
HR     DEPT_ID_PK                           27         27
HR     DEPT_LOCATION_IX                      7         27
HR     EMP_DEPARTMENT_IX                    11        106
HR     EMP_EMAIL_UK                        107        107
HR     EMP_EMP_ID_PK                       107        107
HR     EMP_JOB_IX                           19        107
HR     EMP_MANAGER_IX                       18        106
HR     EMP_NAME_IX                         107        107
HR     JHIST_DEPARTMENT_IX                   6         10
HR     JHIST_EMPLOYEE_IX                     7         10
HR     JHIST_EMP_ID_ST_DATE_PK              10         10
HR     JHIST_JOB_IX                          8         10
HR     JOB_ID_PK                            19         19
HR     LOC_CITY_IX                          23         23
HR     LOC_COUNTRY_IX                       14         23
HR     LOC_ID_PK                            23         23
HR     LOC_STATE_PROVINCE_IX                17         17
HR     REG_ID_PK                             4          4
IX     AQ$_STREAMS_QUEUE_TABLE_Y             0          0
OE     ACTION_TABLE_MEMBERS                132        132
OE     INVENTORY_IX                       1112       1112
OE     INV_PRODUCT_IX                      208       1112
OE     ITEM_ORDER_IX                       105        665
OE     ITEM_PRODUCT_IX                     185        665
OE     LINEITEM_TABLE_MEMBERS              132        132
OE     ORDER_ITEMS_PK                      665        665
OE     ORDER_ITEMS_UK                      665        665
OE     ORDER_PK                            105        105
OE     ORD_CUSTOMER_IX                      47        105
OE     ORD_ORDER_DATE_IX                   105        105
OE     ORD_SALES_REP_IX                      9         70
OE     PRD_DESC_PK                        8640       8640
OE     PRODUCT_INFORMATION_PK              288        288
OE     PROD_NAME_IX                       3727       8640
OE     PROD_SUPPLIER_IX                     62        288
OE     PROMO_ID_PK                           2          2
PM     PRINTMEDIA_PK                         4          4
SH     CHANNELS_PK                           5          5
SH     COSTS_PROD_BIX                        0          0
SH     COSTS_TIME_BIX                        0          0
SH     COUNTRIES_PK                         23         23
SH     CUSTOMERS_GENDER_BIX                  2          5
SH     CUSTOMERS_MARITAL_BIX                11         18
SH     CUSTOMERS_PK                      55500      55500
SH     CUSTOMERS_YOB_BIX                    75         75
SH     FW_PSC_S_MV_CHAN_BIX                  4          4
SH     FW_PSC_S_MV_PROMO_BIX                 4          4
SH     FW_PSC_S_MV_SUBCAT_BIX               21         21
SH     FW_PSC_S_MV_WD_BIX                  210        210
SH     PRODUCTS_PK                          72         72
SH     PRODUCTS_PROD_CAT_IX                  5         72
SH     PRODUCTS_PROD_STATUS_BIX              1          1
SH     PRODUCTS_PROD_SUBCAT_IX              21         72
SH     PROMO_PK                            503        503
SH     SALES_CHANNEL_BIX                     4         92
SH     SALES_CUST_BIX                     7059      35808
SH     SALES_PROD_BIX                       72       1074
SH     SALES_PROMO_BIX                       4         54
SH     SALES_TIME_BIX                     1460       1460
SH     TIMES_PK                           1826       1826

61 rows selected.

SQL> 

```




