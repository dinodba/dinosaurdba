---
layout: post
title:  "Oracle APEX + ORDS"
date:   2019-02-19 19:38:03 +0000
permalink: /apexords
---

So you could make a strong argument that since this post involves Oracle and the Oracle database that it's prehistoric and I'm not helping my case here. After all, I've been working with the Oracle database for almost 20 years and I'm supposed to be learning about cool technology that developers don't hate, right? However this post is also about ORDS - Oracle REST Data Services. REST - I've already got a blog post about REST so it must be cool! Also, I've actually done something that might be useful to other dinosaur DBAs as I've looked before and never found a good, concise blog post of how to set this all up.

#### Oracle Database
I'm going to assume we all know what an Oracle database is. And I'm also going to assume you're able to install and create one - that's not what this blog is about.

#### Application Expres (APEX)
Oracle Application Express (APEX) is a low-code development platform that enables you to build stunning, scalable, secure apps with world-class features that can be deployed anywhere. That must be true because I copied it straight from the [APEX website](https://www.oracle.com/database/technologies/appdev/apex.html). At my work the DBAs have been using it for many years to host a small app with details about our databases and supporting infrastructure. In fairness it was put together by a much smarter DBA who's now left the organisation and the rest of us try not to break it every time we go near it but it works well and has done for a long time. Elsewhere in the organisation our development teams are using it heavily for a bunch of different services and it's very well received.\
It can be set up with just the database and provide the web facing capability through the database listener. That's how the DBAs run and how our development community started using it.

#### Oracle REST Data Services (ORDS)
From the [website](https://www.oracle.com/database/technologies/appdev/rest.html). Oracle REST Data Services (ORDS) makes it easy to develop modern REST interfaces for relational data in the Oracle Database and the Oracle Database 18c JSON Document Store. A mid-tier Java application, ORDS maps HTTP(S) verbs (GET, POST, PUT, DELETE, etc.) to database transactions and returns any results formatted using JSON.\
Yeah, we don't really know enough yet to fully understand all of that. But what I wanted to learn was how to get APEX running in ORDS so that I don't have to mess around with the database listener. Plus that is the way it should be used in practice. And maybe more importantly I heard at a 19c talk recently that there will be new functionality in the 19c ORDS to make API calls to do a lot of the database actions - this could be very useful for us and I hope to return to that when it's available.

### Steps
I'm running Oracle 12.1.0.2 (non-container) on a VM running RHEL 7. I needed to install Java 8 as root for ORDS:
```
yum install java-1.8.0-openjdk-devel
```

Next I [downloaded](https://www.oracle.com/technetwork/developer-tools/apex/downloads/index.html) APEX and unzipped the file on the same server as the database. This creates an `apex` directory so I copied the file to my `ORACLE_BASE` and unzipped there.
Install the APEX functionality into the database by running the apexins.sql script as SYS:
```
@apexins.sql USERS USERS TEMP /i/
ALTER USER APEX_PUBLIC_USER ACCOUNT UNLOCK;
ALTER USER APEX_PUBLIC_USER IDENTIFIED BY din0saur;
```
Then it's worth creating a new profile with no password lifetime and assigning it to the apex_public_user.

That's it for the APEX installation. The rest of getting it working happens within ORDS.

[Download](https://www.oracle.com/technetwork/developer-tools/rest-data-services/downloads/index.html) ORDS and copy it to the directory you want it to be installed to. ORDS doesn't create an ords directory (like APEX) did so I created one (under `ORACLE_BASE` again) and unzipped the file in there.

We need to backup the APEX images directory. I'm not certain this is required at this stage but it's in the docs and good practice if you upgrade APEX. Back it up with a name representing the current version (in my case, 18.2). The directory is in the apex directory.
```
cp -r images images_18_2
```

Now we want to configure RESTful Services.
```
@apex_rest_config.sql
```
Set passwords

And now to complete the ORDS install:
```
cd $ORACLE_BASE/ords
java -jar ords.war install advanced
Enter the location to store configuration data:$ORACLE_BASE/ords/config
Enter the name of the database server [localhost]:<enter your host details here>
Enter the database listen port [1521]:
Enter 1 to specify the database service name, or 2 to specify the database SID [1]:2
Enter the database SID [xe]:<SID>
Enter 1 if you want to verify/install Oracle REST Data Services schema or 2 to skip this step [1]:1
Enter the database password for ORDS_PUBLIC_USER:
Confirm password:
Requires SYS AS SYSDBA to verify Oracle REST Data Services schema.

Enter the database password for SYS AS SYSDBA:
Confirm password:

Retrieving information.
Enter the default tablespace for ORDS_METADATA [SYSAUX]:USERS
Enter the temporary tablespace for ORDS_METADATA [TEMP]:
Enter the default tablespace for ORDS_PUBLIC_USER [SYSAUX]:USERS
Enter the temporary tablespace for ORDS_PUBLIC_USER [TEMP]:
Enter 1 if you want to use PL/SQL Gateway or 2 to skip this step.
If using Oracle Application Express or migrating from mod_plsql then you must enter 1 [1]:
Enter the PL/SQL Gateway database user name [APEX_PUBLIC_USER]:
Enter the database password for APEX_PUBLIC_USER:
Confirm password:
Enter 1 to specify passwords for Application Express RESTful Services database users (APEX_LISTENER, APEX_REST_PUBLIC_USER) or 2 to skip this step [1]:
Enter the database password for APEX_LISTENER:
Confirm password:
Enter the database password for APEX_REST_PUBLIC_USER:
Confirm password:
Enter 1 if you wish to start in standalone mode or 2 to exit [1]:1
Enter the APEX static resources location:/ora/product/apex/images
Enter 1 if using HTTP or 2 if using HTTPS [1]:
Enter the HTTP port [8080]:
```

Validate the installation
```
java -jar ords.war validate
```

I had trouble accessing APEX so set the APEX password.
```
@apxchpwd.sql
```

And now the APEX admin console is available at:\
http://host:8080/ords/apex_admin

And then you can log into your first APEX workspace at:\
http://host:8080/ords

What you'll find starting ORDS using the command above is that it locks your session. In order to start and stop in the background I used the scripts Tim Hall wrote and made available on his site [here](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-standalone-mode#start-stop).

#### Quick SQL

Something really worth taking a look at is the Quick SQL functionality. This was an app you could add to APEX via the Marketplace but it's now available as standard. I'd never seen this before and it's pretty cool for building the SQL to create your application tables and constraints. You still need to know how to model an application but it'll save a lot of SQL writing once you've done that! Have a look at [this video](https://www.youtube.com/watch?v=Ux2eISE9cSQ) by David Peake.