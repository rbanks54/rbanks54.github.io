---
layout: post
title: How to debug a .NET Core app in Docker with VSCode
date: '2018-07-25T12:00:00.001+10:00'
author: Richard Banks
modified_time: '2018-07-25T12:00:00.001+10:00'
---

Let's assume:
* that you're building an ASP.NET Core web application
* you want to deploy and debug this in a Linux container
* some of your team use Visual Studio 2017+ on Windows
* others want to use Visual Studio Code on Mac
* the entire application is more than just a single web app in a container. You use multiple containers in a composed environment.

This was the situation for a team I worked with recently. People using Macs could do front end work, but struggled with back end changes as they couldn't debug the site in the container if there were problems. This post will explain how we got them working and made them happier.

## Setting the scene

First things first, how do we use docker in our development process?

#### Docker Compose

We use `docker-compose` to bring up our environment.

Our production environment is defined in a `docker-compose.yml` file and we have a `docker-compose.override.yml` file for use when developing locally to tweak the environment for non-prod use.

#### Multi-stage Dockerfile

The ASP.NET Core web container is build with a multi-stage `Dockerfile`. We used the default dockerfile created by the Visual Studio Tools for Docker extension in VS2017 and then tweaked it from there.

#### Debugging from Visual Studio 2017

Windows based team members using Visual Studio 2017 typically have a Visual Studio docker project (.dcproj) in the solution. They can set this as the startup project, hit F5 and get debugging from there. Under the hood Visual Studio generates an extra docker-compose file named `docker-compose.vs.debug.g.yml` and then spins up the containers and starts the site.

How Visual Studio starts the process and attaches the debugger is less clear as the tooling doesn't log anything, however the developer experience is pretty simple.

#### Debugging from VSCode

It's much more manual with VSCode.

We adjusted the web app's `Dockerfile` and added a new target that builds an image we can use for debugging.

We then created a new docker-compose override file that adds features to the container needed for debugging from VSCode. This override file is based on the generated compose file created by the VS2017 tooling.

We also added the `vsdbg` binaries to our git repository so that all developers have the binaries in a well known path. If you've not heard of it, vsdbg is the cross platform debugger that uses a command line to interact with the target application. The [OmniSharp wiki](https://github.com/OmniSharp/omnisharp-vscode/wiki/Attaching-to-remote-processes) explains it a little further.

Developers then manually start the web site in the container using `docker exec` and attach the debugger when they're ready.

## The Development Experience with VSCode
The Dockerfile and other artefacts are shown later in this post, but before looking at them, we should first see the steps we expect people using VSCode to take.

> **Pre-Requisite Step:** We add a variable called DOTNET_PATH to docker's `.env` file. This needs to be configured by each developer based on their platform. As it's a team environment we also ask people to tell git to ignore changes to the file by running `git update-index --assume-unchanged .env`. 

To build, run and debug the app on their local machine, developers do the following:

#### (Re)build the container images

```
docker-compose -f docker-compose.yml -f docker-compose.dev.yml build
```

#### Rebuild the web app
You'll want to make sure it's a debug build

```
dotnet clean MySite.Web  # <-- optional
dotnet build -c debug MySite.Web
```

Where MySite.Web is the sub-folder containing the web site source code.

#### Start the containers

```
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d
```

> At this point the web container will be up, but the web site will not be running. The developer will need to start it when they are ready.

#### Start the web site

We need to execute `dotnet run ...` in the container.

```
docker-compose exec web dotnet --additionalProbingPath /root/.nuget/packages --additionalProbingPath /root/.nuget/fallbackpackages bin/Debug/netcoreapp2.0/MySite.Web.dll
```

> P.S. Yikes! That's long. We recommend you create a shell script to make this simpler to use on a regular basis.

#### Start debugging!

In VSCode go to the debug hub and start the "Attach To MySite.Web (Docker)" debug configuration.

At this point developers can now debug the app as they normally would.


While it might seem a little involved, it becomes very routine once it been done a few times 🙂

## The files that make it all happen

#### docker-compose.yml
First up, the main `docker-compose.yml` file. It looks something like this:

```
version: '3.4'

services:
  web:
    image: MySiteWeb
    build:
      context: .
      dockerfile: MySite.Web/Dockerfile
    environment:
      #...rest of config

  mssql:
    image: microsoft/mssql-server-linux:2017-latest
    hostname: mssql
    #...rest of config

  splunk:
    #...rest of config

  elastic:
    #...rest of config

  #...other containers

#...volumes and networks
```

#### docker-compose.dev.yml

The `docker-compose.dev.yml` file looks like the following:

```
version: '3.4'

services:
  web:
    image: MySiteWeb:dev
    build:
      target: debug
    volumes:
      - ./MySite.Web:/app
      - ./vsdbg:/vsdbg:ro
      - ${HOME}/.nuget/packages:/root/.nuget/packages:ro
      - ${DOTNET_PATH}/sdk/NuGetFallbackFolder:/root/.nuget/fallbackpackages:ro
    environment:
      - DOTNET_USE_POLLING_FILE_WATCHER=1
      - ASPNETCORE_ENVIRONMENT=Docker
    entrypoint: tail -f /dev/null
```

The important things to note in this file are
* the `target:debug` setting in the build options.
    * This ensures we build the debug container image. The Dockerfile is shown below. 
* the `volumes`.
    * We mount our local folder containing the web application (where the `dotnet build` output was placed) as well as the `vsdbg` folder.
    * The binaries for vsdbg can be obtained either from using Visual Studio 2017 (look in the /vsdbg subfolder in your home folder), or by running this shell script `curl -sSL https://aka.ms/getvsdbgsh | bash /dev/stdin -v latest -l ~/vsdbg`
* The two packages related volume mounts
    * When `dotnet build` creates the debug version of the site it only creates DLLs specifically for the web site. No dependencies are placed in the output folder.
    * `dotnet run` will, by default, look for dependencies in the app folder, but since neither the debug build or base container image have those dependencies we need to mount them from the host. This is also why we supply the `--additionalProbingPath` options when we start the site.
* The `entrypoint` override.
    * This simply starts keeps the container up even though it's doing nothing.
    * It's only when we run the `docker-compose exec` statement that the web site actually starts.

#### .env

The `${DOTNET_PATH}` variable specifies the location of the nuget packages for each platform. On windows and mac these are different locations and each developer adjusts this file as necessary.

```
DOTNET_PATH=/usr/local/share/dotnet
#DOTNET_PATH=C:/Program Files/dotnet
```

#### launch.json

We need to tell VSCode how to attach the debugger to the web site in the container process.

Add these settings to `launch.json`

```
{
    "version": "0.2.0",
    "configurations": [
      //...
            {
            "name": "Attach to MySite.Web (Docker)",
            "type": "coreclr",
            "request": "attach",
            "sourceFileMap": {
                "/app": "${workspaceRoot}"
            },
            "processId" : "${command:pickRemoteProcess}",
            "pipeTransport": {
                "debuggerPath": "/vsdbg/vsdbg",
                "pipeProgram": "docker",
                "pipeCwd": "${workspaceRoot}",
                "quoteArgs": false,
                "pipeArgs": [
                    "exec -i MySite_web_1"
                ]
             }
        }
    ]
}
```

Be aware that in the `pipeArgs` property, the container name specified is based on the name of the environment used by docker compose. By default this will be the folder name from where you ran docker-compose up. You will need to adjust this for your specific needs.

#### MySite.Web's Dockerfile

The dockerfile has one specific target added to it to support the approach we're using here:

```
#Target for use with VSCode  - debug build
FROM microsoft/aspnetcore-build:2.0 AS debug

WORKDIR /app
EXPOSE 7000
```


## It wasn't all plain sailing

I originally had trouble getting this working as I was using the microsoft/aspnetcore:2.0 base image (which is a runtime image), however this image is so slimmed down that Linux shell command  `ps` has been removed. Without this command available, the OmniSharp debugger cannot iterate the running process ids in the container and the attach fails.

On Mac, people need to make the location of the dotnet packages folder (/usr/local/share/dotnet) available in their Docker for Mac preferences so that it can be mounted in the container.

We've been unable to configure the launch command in VSCode successfully. Attempting to launch the app directly into the container will call `dotnet run` with the name of the `program` from the config settings. What we couldn't manage to do was get the launch command to respect the `--additionalProbingPath` parameters, specified as either `args` or environment (`env`) options. We tried all sorts of variations of the config, as shown here, without success.

```
  "program": "bin/Debug/netcoreapp2.0/MySite.Web.dll",
  "env": {
      "DOTNET_PACKAGES": "~/.nuget/fallbackpackages"
  },
  "args": ["--additionalProbingPath ~/.nuget/packages"],
```

It will likely need changes in the OmniSharp plug in, but we haven't chased that down just yet.
