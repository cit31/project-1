## Setup Nginx + Tomcat + MariaDB

### Install MariaDB and Configure.

Installation :

```
# yum install mariadb-server -y
# systemctl enable mariadb
# systemctl start mariadb
```

Load Schema:

```
# vi student.sql
create database if not exists studentapp;
use studentapp;
CREATE TABLE if not exists Students(student_id INT NOT NULL AUTO_INCREMENT,
	student_name VARCHAR(100) NOT NULL,
        student_addr VARCHAR(100) NOT NULL,
	student_age VARCHAR(3) NOT NULL,
	student_qual VARCHAR(20) NOT NULL,
	student_percent VARCHAR(10) NOT NULL,
	student_year_passed VARCHAR(10) NOT NULL,
	PRIMARY KEY (student_id)
);
grant all privileges on studentapp.* to 'student'@'%' identified by 'student@1';
flush privileges;
```

```
# mysql < student.sql
```

### Install Tomcat and Configure.

Installation:

```
# useradd studentapp
# su - studentapp
$ wget -qO- http://mirrors.fibergrid.in/apache/tomcat/tomcat-9/v9.0.12/bin/apache-tomcat-9.0.12.tar.gz | tar -xf
$ cd /home/studentapp/apache-tomcat-9.0.12
$ rm -rf webapps/*
$ wget https://github.com/cit31/project-1/raw/master/student.war -O webapps/student.war
$ wget https://github.com/cit31/project-1/raw/master/mysql-connector-java-5.1.40.jar -O lib/mysql-connector-java-5.1.40.jar
```

Configuration:

Update `context.xml` file in the following way.

```
<Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
               maxTotal="100" maxIdle="30" maxWaitMillis="10000"
               username="student" password="student@1" driverClassName="com.mysql.jdbc.Driver"
               url="jdbc:mysql://10.142.0.5:3306/studentapp"/>
```

Note: Ip address `10.142.0.5` is the address of MariaDB Server.

Service Start:

```
$ /home/studentapp/apache-tomcat-9.0.12/bin/startup.sh
```

### Install Tomcat and Configure.

Installation:

```
# yum install nginx -y
# systemctl enable nginx
```

Configuration:

Update the configuration which looks like as follows.

```
# vi /etc/nginx/nginx.conf
location / {
          proxy_set_header X-Forwarded-Host $host;
          proxy_set_header X-Forwarded-Server $host;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_pass http://10.142.0.3:8080/;
        }

```
Note: Ip address `10.142.0.3` is the ip address of the Tomcat server.

Service Start:

```
# systemctl restart nginx
```

