To Run pg_stat_monitor along with EnterpriseDB(a.k.a EPAS) in a Linux environment, I described here How to compile pg_stat_monitor from source code.



# Steps

### 1.Download source code 

### 2.Setting Environments

### 3.Compile

### 4.Verification & Testing





### 1.Download Source Code

#### 1-1. download from git

```
$ git clone git://github.com/percona/pg_stat_monitor.git
```

![image-20231220112454664](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231220112454664.png)



#### 1-2. download by manually

Or You can visit the link below and download it by manually.

https://github.com/percona/pg_stat_monitor.git

After downloading it by Manu, Please extract the zip file in some directory where to compile it.

```
run as root.

$ mkdir pg_stat_monitor
$ cd pg_stat_monitor

-- Download here from website (If you download other directory Please copy here)

$ unzip pg_stat_monitor-main.zip
```





### 2.Setting Environments

#### 2-1. check enterprisedb

To get path of EPAS , You can get it command below.

```
ps -ef |grep postmaster
```

![image-20231220113847463](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231220113847463.png)

#### 2-2. Add Path  in .bash_profile

Add to the .bash_profile of a root account the path environment variable that has been obtained from the above result.

(Because when compiled pg_stat_monitor , It need to pg_config files )

```
vi ~/.bash_profile
```

![image-20231221010035982](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221010035982.png)

After save .bash_profile You have to run again like command

```
$. ./.bash_profile
```



#### 2-3. Change the top_srcdir vaules in the Makefile (*IMPORTANT)

Move to the directory of extract zip files which has been downloaded from git.

Open Makefile in the directory and you can see the variable top_srcdir

```
$vi Makefile
```

![image-20231221005848482](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221005848482.png)

Set the variable for top_srcdir and Its top source directory of EPAS. So You can give this value for the installation directory of EPAS

For example, in steps 2-1 , You were got /usr/edb/as15/bin. So Please give /usr/edb/as15 without bin directory.

See Sample below

![image-20231221010815126](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221010815126.png)

   

### 3.Compile

#### 3-1.Comple the pg_stat_monitor

```
$ cd pg_stat_monitor
$ make USE_PGXS=1
$ make USE_PGXS=1 install
```



If you compiled success You can see the figure below.

![image-20231221012328590](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221012328590.png)

![image-20231221012508293](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221012508293.png)



#### 3-2. Error while compiling

* If your system does not installed 'make' You can see the figure below. (Do install make)

![image-20231221011049708](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221011049708.png)



* If your system does not installed gcc You able to see figure below (Do install gcc)

  ![image-20231221011357787](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221011357787.png)

* If you see the below message Please do run the command as root like below. (Do install redhat-rpm-config)

![image-20231221011633253](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221011633253.png)

```
yum install -y redhat-rpm-config -y
```

* if you see the below message while compile Please do run the command as root like below

  ![image-20231221011931167](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221011931167.png)

```
yum install -y clang
```



### 4.Configuration & Testing

If pg_stat_monitor has been compiled successfully, You can see the pg_stat_monitor files for each directory

```
cd /usr/edb/as15/share/extension
ls -al pg_stat_monitor*
```

![image-20231221013135126](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221013135126.png)

```
cd /usr/edb/as15/lib
ls -al pg_stat_monitor.so
```

![image-20231221013237457](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221013237457.png)



4-1. Configuration

Connect to psql and modify the shared_preload_libraries parameter using the ALTER SYSTEM command.

```
ALTER SYSTEM SET shared_preload_libraries = 'pg_stat_monitor';
```

![image-20231221013825430](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221013825430.png)

Start or restart the EPAS instance to apply the changes

```
systemctl restart edb-as-15
```



Create the extension view with the user that has the privileges of a superuser or a database owner. Connect to psql as a superuser for a database and run the CREATE EXTENSION command:

```
CREATE EXTENSION pg_stat_monitor;
```

![image-20231221014053813](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221014053813.png)

After the setup is complete, you can see the stats collected by pg_stat_monitor.

By default, pg_stat_monitor is created for the postgres database. To access the statistics from other databases, you need to create the extension view for every database.



4-2.Testing

Instead of supplying one set of ever-increasing counts, pg_stat_monitor computes stats for a configured number of time intervals; time buckets. This allows for much better data accuracy, especially in the case of high-resolution or unreliable networks.

```
SELECT bucket, bucket_start_time, query,calls FROM pg_stat_monitor ORDER BY bucket;
```

![image-20231221014119727](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221014119727.png)



pg_stat_monitor provides additional metrics for detailed analysis of query performance from various perspectives, including client connection details like user name, application name, IP address to name a few relevant columns. With this information, pg_stat_monitor enables users to track a query to the originating application. More details about the application or query may be incorporated in the SQL query in a Google’s Sqlcommenter format.

```
SELECT userid,  datname, queryid, substr(query,0, 50) AS query, calls FROM pg_stat_monitor;
```

![image-20231221014233292](/Users/hyungjoonkim/Library/Application Support/typora-user-images/image-20231221014233292.png)



For more Information , Please visit https://docs.percona.com/pg-stat-monitor/user_guide.html