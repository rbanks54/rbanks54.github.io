---
layout: post
title: How to connect to a Windows SQL Server running in Docker
date: '2017-03-02T12:00:00.001+10:00'
author: Richard Banks
tags:
 - docker
modified_time: '2017-03-02T12:00:00.001+10:00'
---

This is just a quick one and if it helps you, that's great!

I was asked for a little help in connecting to SQL Server running in a Docker windows container on Windows 10. Here's the steps.
Note: I'm assuming you already have Docker for Windows installed and have switched to Windows Server containers.

1. Pull the image from the Docker repository

```
docker pull microsoft/sql-server-windows
```

2. Start a container instance

```
docker run -d -p 1433:1433 -e sa_password={my password} -e ACCEPT_EULA=Y --name sql microsoft/mssql-server-windows
```

If you want persistence of data in your container you'll need to mount a volume. Here's the variation for that

```
docker volume create sql-data 
REM ^^^ You only need to do this once ^^^

docker run -d -p 1433:1433 --name sql -v sql-data:C:/temp/ -e attach_dbs="[{'dbName':'MyDb','dbFiles':['C:\\temp\\mydb.mdf','C:\\temp\\mydb_log. ldf']}]" -e sa_password={my password} -e ACCEPT_EULA=Y microsoft/mssql-server-windows
```

Be aware of one thing with regards to ports, if port 1433 is in use locally (maybe becuase you're also running a local SQL server instance) you might need to use a different host port number. For example `-p 11433:1433`

3. Check that it works

Let's run sqlcmd directly on the container.

```
docker exec -it sql sqlcmd
```

You should launch directly into the SQL Command Line utility and be able to see your user information

```
1> print Suser_Sname()
2> GO
User Manager\ContainerAdministrator
```

4. Connect from your desktop

On Windows 10, you can't just connect to windows containers via localhost. You'll need to know the ip address that the container is listening on. In a future O/S update this should be fixed, but for now a design restriction with networking and hyper-v prevents it working properly.

For now do this:

```
docker inspect --format '{{"{{.NetworkSettings.Networks.nat.IPAddress" }}}}' sql
```

This will tell you the IP address the container is listening on.

Now open SQL Server Management Studio, Visual Studio or SqlCmd

Here's what you should see (using SqlCmd):

```
c:\pd>sqlcmd -U sa -S 172.28.129.7
Password:
1> print Suser_Sname()
2> GO
sa
1>
```

Note that we're operating under a different account since we logged in from externally.

Sweet! That's it. Nice and simple.