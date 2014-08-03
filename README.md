##Configure JDBCRealm JAAS for mysql and Jboss 7 with form based authentication

####Create database 
```
CREATE DATABASE tutorialsdb;

DROP DATABASE IF EXISTS tutorialsdb;
CREATE DATABASE tutorialsdb;
USE tutorialsdb;
CREATE TABLE users (
username varchar(20) NOT NULL PRIMARY KEY,
password varchar(20) NOT NULL
);
CREATE TABLE roles (
rolename varchar(20) NOT NULL PRIMARY KEY
);
CREATE TABLE users_roles (
username varchar(20) NOT NULL,
rolename varchar(20) NOT NULL,
PRIMARY KEY (username, rolename),
CONSTRAINT users_roles_fk1 FOREIGN KEY (username) REFERENCES users (username),
CONSTRAINT users_roles_fk2 FOREIGN KEY (rolename) REFERENCES roles (rolename)
);
INSERT INTO `tutorialsdb`.`users` (`username`, `password`) VALUES ('admin', 'admin');
INSERT INTO `tutorialsdb`.`roles` (`rolename`) VALUES ('user');
INSERT INTO `tutorialsdb`.`users_roles` (`username`, `rolename`) VALUES ('admin', 'user');
COMMIT;
```
####Creating module for mysql :

Navigate to `<jboss_home>/modules/com`  e.g. `C:\jboss-as-7.1.1.Final\modules\com`
Create a folder called `mysql` in it and under `mysql` , create another folder named `main`
Under `main` , copy your mysql connector jar file  and create a file called `module.xml`

```
	<module xmlns="urn:jboss:module:1.1" name="com.mysql">	 
		<resources>
			<resource-root path="mysql-connector-java-5.1.23-bin.jar"/>
		</resources>
		<dependencies>
			<module name="javax.api"/>
			<module name="javax.transaction.api"/>
			<module name="javax.servlet.api" optional="true"/>
		</dependencies>
	</module>
```
####Configure module in standalone.xml
Navigate to `<jboss_home>/standalone/configuration` and open `standalone.xml`

You will find a `datasources` tag under which you need to put this
```
	 <datasource jta="false" jndi-name="java:/jBossJaasMysql" pool-name="jBossJaasMysql" enabled="true" use-ccm="false">
		<connection-url>jdbc:mysql://localhost:3306/tutorialsdb</connection-url>
		<driver-class>com.mysql.jdbc.Driver</driver-class>
		<driver>mysql</driver>
		<security>
			<user-name>root</user-name>
			<password>root</password>
		</security>
		<validation>
			<validate-on-match>false</validate-on-match>
			<background-validation>false</background-validation>
		</validation>
		<statement>
			<share-prepared-statements>false</share-prepared-statements>
		</statement>
	</datasource>
```				
jndi name is the identifier we are going to use in our security configuration.
`jdbc:mysql://localhost:3306/tutorialsdb` is our database to which jndi name points


Add following code to subsystems tag.

```
	<subsystem xmlns="urn:jboss:domain:jpa:1.0">
				<jpa default-datasource="java:/jBossJaasMysql"/>
	</subsystem>
```

Add driver as module

```
	<driver name="mysql" module="com.mysql">
		<xa-datasource-class>com.mysql.jdbc.Driver</xa-datasource-class>
	</driver>
```

Add following code to  `standalone.xml`  under `security-domains`


```
	<security-domain name="jBossJaasMysqlRealm">
		<authentication>
			<login-module code="Database" flag="required">
				<module-option name="dsJndiName" value="java:/jBossJaasMysql"/>
				<module-option name="principalsQuery" value="select password from users where username = ?"/>
				<module-option name="rolesQuery" value="select roleName,'Roles' from users_roles where username=?"/>
			</login-module>
		</authentication>
	</security-domain>
```

Deploy the application and visit 
`http://localhost:8080/jBossJaasMysql/protected/index.jsp`  and enter username as `admin` and password as `admin`.

##Mysql setup

Connect root User
`shell> mysql --user=root --password=root mysql`

Create guard User
`mysql> CREATE USER 'guard'@'localhost' IDENTIFIED BY 'guard';`

Grant all to guard User
`GRANT ALL PRIVILEGES ON *.* TO 'guard'@'localhost' WITH GRANT OPTION;`


Check Grants
`SHOW GRANTS FOR 'guard'@'localhost';`

Drop User
`DROP USER 'guard'@'localhost';`


Check MySql Version
`SELECT VERSION();`

Check Current User
`SELECT CURRENT_USER();`

Create database
`CREATE DATABASE guard;`

Connect User
`shell > mysql -h localhost -u guard -p guard`

Run explain plan
`EXPLAIN <SQL Query>;`

Find indexes on table
`INDEX IN <table_name>;`
