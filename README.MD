# **How to install and configure PostgreSQL on a CentOS machine**

# Table of Contents


<br></br>
## Step 1: Login as root and update system  
---
The first thing we want to do is make sure we are logged in as root. To do that in CentOS we type 

`$ su`

Then To update the system we use the following command:

`$  sudo yum update`

The system is now ready to install PostgreSQL

<br></br>
<br></br>
<br></br>
## Step 2: Install PostgreSQL
---
If we didn't know our machine had postgres or not, we could simply check by running the following command:

`$  dnf module list postgresql`

If you are reading this guide though chances are that you do not have it installed. Currently the most recent version of postgres is 14, so we can install it using this command:

`$ sudo dnf module enable postgresql:14`

<br></br>
<br></br>
<br></br>
## Step 3: Install PostgreSQL Server
---
Now we install the postgres server:

`$ sudo dnf install postgresql-server`

Once the server is installed the database needs to be initalized

`$  postgresql-setup --initdb`

**To enable start at system boot, use the following commands**

`$ systemctl enable postgresql`

`$ systemctl start postgresql`


To check the status of your PostgreSQL service, use the following command:

`$ systemctl status postgresql`

If it says 'active' then it is running

<br></br>
<br></br>
<br></br>
## Step 4: Configuring Database
---

To create a password for the **postgres** account

`# passwd postgres`

This will ask you to type your password two times. 

After setting up the password, we can log into the **postgres** account using su:

`$ s -i -u postgres`

<br></br>
<br></br>
<br></br>
## Step 5: Creating a New Role 

We want to create a non-root role so that they will not have root privileges. Type the following to start a create user prompt inside of the Postgres session

`postgres@server:~$ createuser --interactive`

If you want to do it via sudo you would type 

`$ sudo -u postgres createuser --interactive`

Here the system will ask you the role name and the type for the new user. 

```
Enter name of role to add: test
Shall the new role be a superuser? (y/n) "y"
```

<br></br>
<br></br>
<br></br>


## Step 6: Creating New Database
Postges' authentication makes it so that for any role used to log in, that role will also have a database with the same name. 
For example we created the test user, so to create a new database within PostgreSQL, use the following command:

`createdb databasename test`

<br></br>
**note** 
If you want to create this with sudo outside of postgres you would type the following command:

`$ sudo -u postgres createdb test`


<br></br>
<br></br>
<br></br>

## Step 7: Access the PostgreSQL Prompt With the New Role
---

Once we have created a non-root role, we have to create a user with the same name as the role and database.

If you don't have a ID available, you can type:

`$ sudo adduser test`

Log in as the new user 

`$ sudo -i -u test`

`$ psql`

If you are connecting to a current database you would type 

`$ psql -d postgres`

Once logged in you can check your current connection by typing:

`test=# \conninfo`

<br></br>
<br></br>
<br></br>


## Step 8: Creating Table & Inserting Data
---
There are several ways to insert data into the database. In this readme I have included two ways on how to do so; Postgres and Python. 


### Native Postgres
Lets create a new table to store some data. The syntax to add fields in the new table is looks like:

>CREATE TABLE table_name (
>
>    column_name1 col_type (field_length),
>    
>    column_name2 col_type (field_length),
>    
>    column_name3 col_type (field_length)
>
>);

For example we can create a sample table

>CREATE TABLE playground (
>    equip_id serial PRIMARY KEY,
>    
>    type varchar (50) NOT NULL,
>    
>    color varchar (25) NOT NULL,
>    
>    location varchar(25) check (location in ('north', 'south', 'west', 'east', 'northeast', 'southeast', 'southwest', 'northwest')),
>    
>    install_date date
>);

you can see your new table by typing

>sammy=# \d

Now we can insert some data. Lets add a slide and swing to the playground


>INSERT INTO playground (type, color, location, install_date) VALUES ('slide', 'blue', 'south', '2017-04-28');

>INSERT INTO playground (type, color, location, install_date) VALUES ('swing', 'yellow', 'northwest', '2018-08-16');


<br></br>
<br></br>
<br></br>
### Python 
If we want to insert data using python, we need to install a module called psycopg2. It is usually installed but if you do not have it you can install it using

`$ yum install python-psycopg2`

To connect, we must know four things about the PostgreSQL server.

<ul>
<li> Username: The username for PostgreSQL</li>
<li> Password: The password chosen by the user</li>
<li> Host Name: The IP where PostgreSQL is running</li>
<li> Database Name: The name of the database we created</li>

Now we open a text editor and create a connection to the database

```
#!/usr/bin/python
import psycopg2

conn = psycopg2.connect(database="test", user="test", password="123", host="127.0.0.1", port="5432")
```
we can add to this file and create a table 

```

import psycopg2
from psycopg2 import Error

try:
    connection = psycopg2.connect(user="test",
                                  password="123",
                                  host="127.0.0.1",
                                  port="5432",
                                  database="test")

    cursor = connection.cursor()
    # SQL query to create a new table
    create_table_query = '''CREATE TABLE mobile
          (ID INT PRIMARY KEY     NOT NULL,
          MODEL           TEXT    NOT NULL,
          PRICE         REAL); '''
    # Execute a command: this creates a new table
    cursor.execute(create_table_query)
    connection.commit()
    print("Table created successfully in PostgreSQL ")

except (Exception, Error) as error:
    print("Error while connecting to PostgreSQL", error)
finally:
    if connection:
        cursor.close()
        connection.close()
        print("PostgreSQL connection is closed")
```

<br></br>
<br></br>
<br></br>
## Step 9: Automating Backup of Database
---

To perform a backup with this utility, just call it and specify the destination file

`
su - postgres 
pg_dump -U postgres db_name | gzip > backup_file.gz
`

To automate it we

1. Set the postgresql user
    
    `su - postgres`
2. Create a backup storage folder
   
    ```mkdir -p ~/postgres/backups```

3. Open crontab by running the command 
   
    ```crontab -e```
4. Add this line at the end
   
    ``` 0 0 * * * pg_dump -U postgres db_name | gzip > backup_file.gz```


Of course postgres and db_name should be swapped with the non-root user and their database name

## Step 10: Setting Up Media Wiki

Install prereq packages

`
$ yum install httpd php php-hd php-xml php-intl
`

Download Media Wiki 1.37.0

`
wget https://releases.wikimedia.org/mediawiki/1.37.0/mediawiki-1.37.0.tar.gz
`

Untar the file

`
tar -zxpvf mediawiki-1.37.0.tar.gz
`

Move media wiki to its document root (/var/www/html/mediawiki/)

`
mv mediawiki-1.37.0 /var/www/html/mediawiki
`

set the required permission on the directory

`chown -R apache:apache /var/www/html/mediawiki/
chmod 755 /var/www/html/mediawiki/
`

If your system is running behind a firewall, use the commands below to open port 80 for mediawiki



```
$firewall-cmd --zone=public --add-port=80/tcp
--permanent

$ firewall-cmd--reload
success

$ iptables-save | grep 80
A IN_public_allow -p tcp -m tcp --dport 80 -m conntrack --ctstate NEW -j
ACCEPT
```


Now open up Media Wiki, Open the web browser 

type http://<IP_Address_of_System>/mediawiki

![MediaWiki-Web-installation](/assets/MediaWiki-Web-installation.webp "MediaWiki-Web-installation")
Replace the ip address as per your setup . Click on “Set up the wiki”

***Define the Language for MediaWiki***
![MediaWiki_language](/assets/MediaWiki_language.webp "MediaWiki_language")

Click on Continue

***MediaWiki Environment Setup Check:***
 
![mediawiki-setup-check](/assets/mediawiki-setup-check.webp)

Click on Continue

***Define the Mediawiki database, database user name and Password.***

![MediaWiki-database-username](/assets/MediaWiki-database-username.webp)

***Media Wiki Database Settings***

![MediaWiki-database-engine](/assets/MediaWiki-database-engine.webp)

***Define Name of Wiki and its Administrator Account***

![Mediawik-admin-account](/assets/Mediawik-admin-account.webp)

Click on Continue to finish the installation

![click_continue_to_finish_installation](/assets/click_continue_to_finish_installation.webp)

MediaWiki installation is completed. Now Download “LocalSettings.php” file and placed it under the mediawiki document root (/var/www/html/mediawiki/ )

![mediawiki-installation-completed](/assets/mediawiki-installation-completed.webp)

Now visit the Main page ( http://192.168.1.14/mediawiki/index.php/Main_Page) and Login Admin Panel of Mediawiki

![mediawiki-main-page](/assets/mediawiki-main-page.webp)

Click on Log In . Enter the Admin Credentials that we have set during the installation.

![Log-in-Mediawiki-admin](/assets/Log-in-Mediawiki-admin.webp)

![mediawiki-admin-panel-page](/assets/mediawiki-admin-panel-page.webp)


<br></br>
<br></br>
<br></br>
## Step 11: Backing Up Media Wiki

Similar to step 9, 

To perform a backup with this utility, just call it and specify the destination file

`
su - postgres 
pg_dump mywiki > mywikidump.sql
`

To automate it we

1. Set the postgresql user
    
    `su - postgres`
2. Create a backup storage folder
   
    ```mkdir -p ~/postgres/backups```

3. Open crontab by running the command 
   
    ```crontab -e```
4. Add this line at the end
   
    ```0 0 * * * pg_dump -U postgres db_name | mywiki > mywikidump.sql```



