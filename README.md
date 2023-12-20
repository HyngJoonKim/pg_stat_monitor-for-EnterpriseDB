# pg_stat_monitor-for-EnterpriseDB
pg_stat_monitor for EnterpriseDB





To Run pg_stat_monitor along with EnterpriseDB(a.k.a EPAS) in a Linux environment, I described here How to compile pg_stat_monitor from source code.



# Steps

### 1. Download source code 

### 2.Setting Environments

### 3.Compile

### 4. Verification & Testing





### 1.Download Source Code

#### 1-1. download from git

```
$ git clone git://github.com/percona/pg_stat_monitor.git
```

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/30945e21-230f-4148-9790-ae91f72369d2)




#### 1-2. download by manually

Or You can visit the link below and download it by manually.

https://github.com/percona/pg_stat_monitor.git

After downloading it by Manu, Please extract the zip file in some directory where to compile it.

```
run as root.

$ mkdir pg_stat_monitor
$ cd pg_stat_monitor

-- Download here from the website (If you download other directories Please copy it here)

$ unzip pg_stat_monitor-main.zip
```





### 2.Setting Environments

#### 2-1. check enterprisedb

To get the path of EPAS, You can get it command below.

```
ps -ef |grep postmaster
```

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/ddb90df2-c88b-4c45-96b2-64fcfce19fb6)


#### 2-2. Add Path  in .bash_profile

Add to the .bash_profile of a root account the path environment variable that has been obtained from the above result.

(Because when compiled pg_stat_monitor , It need to pg_config files )

```
vi ~/.bash_profile
```

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/b5854471-dc74-4ae6-807e-288a893f0bcc)


After saving .bash_profile You have to run it again like the command

```
$. ./.bash_profile
```



#### 2-3. Change the top_srcdir values in the Makefile (*IMPORTANT)

Move to the directory of extract zip files which has been downloaded from git.

Open Makefile in the directory and you can see the variable top_srcdir

```
$vi Makefile
```

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/3864638a-f672-4b0d-813e-8f621d1da4d0)


Set the variable for top_srcdir and Its top source directory of EPAS. So You can give this value for the installation directory of EPAS

For example, in steps 2-1 , You were got /usr/edb/as15/bin. So Please give /usr/edb/as15 without bin directory.

See Sample below

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/9a6ee8fb-1476-4b9a-8b2d-2e189375c60d)


   

### 3. Compile

#### 3-1.Comple the pg_stat_monitor

```
$ cd pg_stat_monitor
$ make USE_PGXS=1
$ make USE_PGXS=1 install
```



If you compiled success You can see the figure below.

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/e7bb7c0b-9fcf-4c63-8234-257db289b4bd)


![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/b618bd06-abd2-48ca-adb2-73c05f559bfd)




#### 3-2. Error while compiling

* If your system does not install 'make' You can see the figure below. (Do install make)

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/2b05e6e2-acb7-4574-a5a7-3d2c14cdf984)




* If your system does not install gcc You are able to see the figure below (Do install gcc)

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/30092f28-704d-478d-8c72-dba4949adb75)


* If you see the below message Please do run the command as root like below. (Do install redhat-rpm-config)

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/5b915ac9-6b03-47a0-9d9c-91d3324a0ad8)


```
yum install -y redhat-rpm-config -y
```

* if you see the below message while compiling Please do run the command as root like below

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/1bfe4614-f25a-4c0a-a72b-12e47984753d)


```
yum install -y clang
```



### 4. Configuration & Testing

If pg_stat_monitor has been compiled successfully, You can see the pg_stat_monitor files for each directory

```
cd /usr/edb/as15/share/extension
ls -al pg_stat_monitor*
```

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/2df7fd19-e5f7-4d80-9577-fd71da669f9e)


```
cd /usr/edb/as15/lib
ls -al pg_stat_monitor.so
```

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/f1844d63-6945-4ce9-8206-ab9ffb1ad5aa)




#### 4-1. Configuration

Connect to psql and modify the shared_preload_libraries parameter using the ALTER SYSTEM command.

```
ALTER SYSTEM SET shared_preload_libraries = 'pg_stat_monitor';
```

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/a76f9a30-474e-42b2-bfd4-a74bdd757890)


Start or restart the EPAS instance to apply the changes

```
systemctl restart edb-as-15
```



Create the extension view with the user that has the privileges of a superuser or a database owner. Connect to psql as a superuser for a database and run the CREATE EXTENSION command:

```
CREATE EXTENSION pg_stat_monitor;
```

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/6aec1c65-5e3c-4338-b97c-5bd140333552)


After the setup is complete, you can see the stats collected by pg_stat_monitor.

By default, pg_stat_monitor is created for the EPAS database. To access the statistics from other databases, you need to create the extension view for every database.



4-2.Testing

Instead of supplying one set of ever-increasing counts, pg_stat_monitor computes stats for a configured number of time intervals; time buckets. This allows for much better data accuracy, especially in the case of high-resolution or unreliable networks.

```
SELECT bucket, bucket_start_time, query,calls FROM pg_stat_monitor ORDER BY bucket;
```

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/ae4cdd42-fdb8-468d-a90a-53df2ea2f63f)




pg_stat_monitor provides additional metrics for detailed analysis of query performance from various perspectives, including client connection details like user name, application name, IP address to name a few relevant columns. With this information, pg_stat_monitor enables users to track a query to the originating application. More details about the application or query may be incorporated in the SQL query in a Google’s Sqlcommenter format.

```
SELECT userid,  datname, queryid, substr(query,0, 50) AS query, calls FROM pg_stat_monitor;
```

![image](https://github.com/HyngJoonKim/pg_stat_monitor-for-EnterpriseDB/assets/15187312/fccf38a0-3a16-4998-acde-0dec7f1ab18c)




For more information , please visit https://docs.percona.com/pg-stat-monitor/user_guide.html
