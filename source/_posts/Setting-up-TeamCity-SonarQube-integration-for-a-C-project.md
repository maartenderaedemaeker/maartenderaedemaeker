---
title: 'Setting up TeamCity SonarQube integration for a C# project'
tags:
  - Automation
  - SonarQube
  - Code quality
  - Teamcity

categories:
  - Software development
permalink: setting-up-teamcity-sonarqube-integration-for-a-csharp-project
id: 59d3e06faf4f1298d16e0ec2
updated: '2017-08-26 19:31:51'
date: 2017-07-23 14:55:06
---
This blog post follows my previous post ["Getting started with SonarQube on a C# project"](/2017/07/22/Setting-up-TeamCity-SonarQube-integration-for-a-Csharp-project/).
In my first attempt of integrating TeamCity and SonarQube I tried to use [TeamCity SonarQube plugin](https://confluence.jetbrains.com/display/TW/SonarQube+Integration), but it seems to be abandoned as the specific documentation for the runner this plugin relies on doesn't seem to exist anymore [runner documentation](http://docs.codehaus.org/display/SONAR/Analysis+Parameters).
That's why this post will just use the command-line tools available, with the added bonus advantage of easier portability over different types of build servers.
You can check out the source code analyzed at [github](https://github.com/maartenderaedemaeker/Automated-SecurityTesting-Demo/tree/part1).

# Table of contents

* Step 1: Installing the tools
* Step 2: Setting up a build configuration for SonarQube analysis

# Step 1: Installing the tools

We're going to install some dependencies on the build agent.
First off, [install chocolatey](https://chocolatey.org/install).
After that, install SonarQube runner.
```
choco install -y microsoft-build-tools msbuild-sonarqube-runner
```

# Step 2: Setting up a build configuration

Since this is a new project in TeamCity, I'll start with creating a project.

![Teamcity-SonarQube-3](/content/images/2017/07/Teamcity-SonarQube-3.png)

After clicking through the wizard, TeamCity discovered one build step for my project.
I'd like to start with a clean slate and click on configure build steps manually.

![Teamcity-SonarQube-4](/content/images/2017/07/Teamcity-SonarQube-4.png)

We're going to create 4 build steps.
![Teamcity-SonarQube-5](/content/images/2017/07/Teamcity-SonarQube-5.png)

* Step 1: 
    Runner type: "NuGet Installer"
    This step will restore the NuGet packages required for building the solution.
* Step 2:
    Runner type: "Command Line"
    ```
        SonarQube.Scanner.MSBuild.exe begin /k:"%sonar.project%" 
            /d:"sonar.host.url=%sonar.host.url%" /d:"sonar.login=%sonar.login%" 
            /d:"sonar.organization=%sonar.organization%" /v:"%build.number%"
    ```
    This step is used to hook in the SonarQube runner into the msbuild process.
    You'll notice the parameters enclosed with %'s. More on that later.
* Step 3:
    Runner type: "MSBuild"
    This step will build the solution.
* Step 4:
    Runner type: "Command line"
```
    SonarQube.Scanner.MSBuild.exe end /d:"sonar.login=%sonar.login%"
```
This step will submit the analysis to SonarQube.

After defining these steps we'll also need to define the parameters used in the commands.
Using these allows us to only need to change their value in one place. If you want, you could set this parameters globally at your root project, or for all your builds on this specific project. 
![Teamcity-SonarQube-6](/content/images/2017/07/Teamcity-SonarQube-6.png)

# Step 3: Test the configuration

There we go, now you can press the run build button to see if it works! :-)
If all went well you should now have a basic working build configuration.
Now you can start tweaking the configuration to fit your needs. Good luck!