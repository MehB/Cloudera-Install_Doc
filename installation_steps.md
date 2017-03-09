# Installation Steps

* Careful review of hardware, OS, disk, and network/kernel settings
* Install supported Oracle JDK
* Install/configure database server
* Configure server to customer requirements
* Create databases, connect the CM server to them
* Accessing MySQL requires a JDBC connector
* CM installation (agents)

* Compatibility matrix: https://www.cloudera.com/downloads/manager/5-9-0.html

__________________________________________
### Pre-requierements

* Change root password of each hosts


* Create a new user with passwd (each hosts)


* Add the new user in the sudoers file


* Supported OS

* Disable Firewall


* Disable SELinux:

```
sudo vi /etc/sysconfig/selinux
   SELINUX=disable
```
* Set the `vm.swappiness` to 1:

`$ echo "vm.swappiness = 1" >> /etc/sysctl.conf`


* Disable transparent hugepage support

``
echo never /sys/kernel/mm/transparent_hugepage/defrag
never /sys/kernel/mm/transparent_hugepage/defrag
``

* List your network interface configuration


* Resize root partition

`resi`


* Configure a non-root volume:

`mkfs.ext4 -m 0 /dev/sdx`

`tune2fs -m 0 /dev/sdx`

* Run nscd service

* Run ntpd service

* Change root password of each hosts

* Create a new user with passwd

* SSH between all hosts

* Edit the /etc/hosts of each hosts

* Java installation:

   * Download Oracle JDK 7:
     http://download.oracle.com/otn-pub/java/jdk/7u55-b13/jdk-7u55-linux-x64.tar.gz
```
sudo wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/7u55-b13/jdk-7u55-linux-x64.tar.gz" -O jdk-7u55-linux-x64.tar.gz
```

   * Move JDK download to /usr/java/ and extract it:
```
[hadoop@bigdata03 ~]$ cp jdk-7u55-linux-x64.tar.gz /usr/java/

[hadoop@bigdata03 ~]$ tar xzf jdk-7u55-linux-x64.tar.gz
```

   * Install Java with alternatives:
```
[hadoop@bigdata03 java]$ sudo alternatives --install /usr/bin/java java /usr/java/jdk1.7.0_55/bin/java 2
[hadoop@bigdata03 ~]$ sudo alternatives --config java

There are 4 programs which provide 'java'.

  Selection    Command
-----------------------------------------------
*  1           /usr/lib/jvm/jre-1.7.0-openjdk.x86_64/bin/java
   2           /usr/lib/jvm/jre-1.6.0-openjdk.x86_64/bin/java
 + 3           /usr/java/jre1.7.0_55/bin/java
   4           /usr/java/jdk1.7.0_55/bin/java

Enter to keep the current selection[+], or type selection number: 4
```


   * OPTIONAL (but recommended): Setup javac and jar
```
sudo alternatives --install /usr/bin/jar jar /usr/java/jdk1.7.0_55/bin/jar 2
sudo alternatives --install /usr/bin/javac javac /usr/java/jdk1.7.0_55/bin/javac 2
```
   * Set Jar and Javac:
```
sudo alternatives --set jar /usr/java/jdk1.7.0_55/bin/jar
sudo alternatives --set javac /usr/java/jdk1.7.0_55/bin/javac
```

   * Export $JAVA_HOME:
```
[hadoop@bigdata03 ~]$ vi .bash_profile

export JAVA_HOME=$JAVA_HOME:/usr/java/jdk1.7.0_55
export JRE_HOME=$JRE_HOME:/usr/java/jdk1.7.0_55/jre
```

   * Verify Java Version:
```
[hadoop@bigdata01 ~]$ java -version
java version "1.7.0_55"
Java(TM) SE Runtime Environment (build 1.7.0_55-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.55-b03, mixed mode)
```







__________________________________________
### Databases configuration

https://www.cloudera.com/documentation/enterprise/5-9-x/topics/install_cm_mariadb.html

MariaDB Installation useful link: http://www.tecmint.com/install-mariadb-in-linux/

* Enable the repo to install MySQL/MariaDB 5.5

```
# MariaDB 5.5 CentOS repository list - created 2013-08-11 14:22 UTC
# http://mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/5.5/centos6-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```


* Install MariaDB Server 5.5 on all nodes

   * Install mysql-server on the server and replica nodes (for replication)

`yum update`
`yum -y install MariaDB MariaDB-server`

   * Set password protection for the server
   * Revoke permissions for anonymous users
   * Permit remote privileged login
   * Remove test databases
   * Refresh privileges in memory
   * Refreshes the mysqld service

   ```
   Setting the root password ensures that nobody can log into the MariaDB
   root user without the proper authorisation.

   Set root password? [Y/n] Y
   New password:
   Re-enter new password:
   Password updated successfully!
   Reloading privilege tables..
    ... Success!


   By default, a MariaDB installation has an anonymous user, allowing anyone
   to log into MariaDB without having to have a user account created for
   them.  This is intended only for testing, and to make the installation
   go a bit smoother.  You should remove them before moving into a
   production environment.

   Remove anonymous users? [Y/n] Y
    ... Success!

   Normally, root should only be allowed to connect from 'localhost'.  This
   ensures that someone cannot guess at the root password from the network.

   Disallow root login remotely? [Y/n] N
    ... Success!

   By default, MariaDB comes with a database named 'test' that anyone can
   access.  This is also intended only for testing, and should be removed
   before moving into a production environment.

   Remove test database and access to it? [Y/n] Y
    - Dropping test database...
    ... Success!
    - Removing privileges on test database...
    ... Success!

   Reloading the privilege tables will ensure that all changes made so far
   will take effect immediately.

   Reload privilege tables now? [Y/n] Y
    ... Success!

   Cleaning up...

   All done!  If you've completed all of the above steps, your MariaDB
   installation should now be secure.

   Thanks for using MariaDB!
   ```

* Create Databases for:
   * Activity Monitor
   * Reports Manager
   * Hive Metastore Server
   * Sentry Server
   * Cloudera Navigator Audit Server
   * Cloudera Metadata Server
   * Hue
   * Oozie
   * scm
   * cmf -> Created by the `scm_prepare_database.sh` script (`/usr/share/cmf/schema/scm_prepare_database.sh`)

```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| amon               |
| cmf                |
| hue                |
| metastore          |
| mysql              |
| nav                |
| navms              |
| oozie              |
| performance_schema |
| rman               |
| scm                |
| sentry             |
+--------------------+
```

https://github.com/mitsaravanan/SEBC/issues/1


__________________________________________
### Install Cloudera Manager:

https://united.softserveinc.com/blogs/hadoop-cluster-cloudera-manager/

* Enable cloudera manager repo

* Prepare the Database:

https://www.cloudera.com/documentation/enterprise/5-5-x/topics/cm_ig_installing_configuring_dbs.html
https://www.cloudera.com/documentation/enterprise/5-5-x/topics/install_cm_mariadb.html

*



















__________________________________________
### CM Cluster hosts roles:


    Node 1:
    -------
    HDFS Balancer
    Hive Gateway
    HiveServer2
    Hue Server
    Cloudera Management Service Alert Publisher
    Cloudera Management Service Event Server
    Cloudera Management Service Host Monitor
    Cloudera Management Service Reports Manager
    Cloudera Management Service Service Monitor
    Oozie Server
    YARN (MR2 Included) JobHistory Server

    Node 2:
    -------
    HDFS Failover Controller
    HDFS NameNode
    Hive Gateway
    Hive Metastore Server
    YARN (MR2 Included) ResourceManager

	Node 3:
    -------
    HDFS DataNode
    HDFS Failover Controller
    HDFS JournalNode
    HDFS NameNode
    Hive Gateway
    YARN (MR2 Included) NodeManager
    ZooKeeper Server

	Node 4:
    -------
    HDFS DataNode
    HDFS JournalNode
    Hive Gateway
    YARN (MR2 Included) NodeManager
    ZooKeeper Server

	Node 5:
    -------
    HDFS DataNode
    HDFS JournalNode
    Hive Gateway
    YARN (MR2 Included) NodeManager
    ZooKeeper Server
