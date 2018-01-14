---
title: Adding NUnit code coverage to SonarQube using OpenCover
tags: 
  - SonarQube
  - Code quality
  - C#
  - .NET
  - Testing
categories:
  - Software development
permalink: nunit-opencover-results-in-sonarqube
id: 59fd93b687a49eb687f6fce6
updated: '2017-11-06 07:32:28'
date: 2017-11-04 11:17:26
author: Maarten De Raedemaeker
---
In this blogpost I'll explain how to add your NUnit code coverage to your project analysis results, by using OpenCover.
You can find a sample project you can use to try it out yourself [on my github](https://github.com/maartenderaedemaeker/SonarQubeNunitIntegrationDemo).

* [What is SonarQube](#sonarqube-intro)
* [What is OpenCover](#opencover-intro)
* [Using OpenCover](#using-opencover)
* [Adding test coverage results to SonarQube](#coverage-results)
<a name="sonarqube-intro"></a>
# What is SonarQube
In short, SonarQube is a tool for monitoring your code quality, mainly using static analysis.
For more information, take a look at [my other posts about SonarQube](/tag/sonarqube/) and the [SonarQube documentation](https://docs.sonarqube.org/display/SONAR/Documentation).
<a name="opencover-intro"></a>
# What is OpenCover
OpenCover is a code coverage tool.
It allows us to run an application using the tool, and it will monitor what parts of the code are executed during the execution. 
We can use it to run our unit test framework, and it will tell us how much of our code is covered by our tests.

In this blogpost, I'll be using NUnit as my unit testing framework.
<a name="using-opencover"></a>
# Using OpenCover
In my solution, I've added the OpenCover and NUnit.ConsoleRunner nuget packages.
If I take a look in my packages folder, I can find the corresponding packages.

![2017-11-05-09_13_58-packages](/images/2017/11/2017-11-05-09_13_58-packages.png)

We are interested in the following two executables included in these packages:
* `{OpenCoverFolder}\tools\OpenCover.Console.exe`
* `{NUnitConsoleFolder}\tools\nunit3-console.exe`

As I explained earlier, we're going to use OpenCover to run the NUnit console runner to measure our code coverage.

We can do it by executing the following command: (replace the variables with the real values)
```
{PATH_TO_OpenCover.Console.exe} "-target:{PATH_TO_nunit3-console.exe}" 
"-targetargs:{PATH_TO_OUR_TEST_ASSEMBLIES}" 
"-register:user" "-output:opencover.xml"
```
Let's take a closer look at the command:
* We run the `OpenCover.Console.exe` executable
* We are telling it to use the nunit3-console.exe as it's target using the `-target` parameter. 
* We use the `-targetargs` parameter to pass through arguments to nunit3-console.exe. (for example in my case, I would pass through `-targetargs:.\Demo.Tests\bin\Debug\Demo.Tests.dll`, this would tell NUnit to run all tests in this assembly. 
* We use the `-register:user` argument to tell OpenCover to register the agent process it uses for the current user. (otherwise you'll get an empty code coverage result)
* We specify the `-output` parameter to choose a file where the OpenCover results get written to.

When running the command, we can see NUnit running the test and the code coverage results being written.

![2017-11-05-09_31_16-Cmder](/images/2017/11/2017-11-05-09_31_16-Cmder.png)

<a name="coverage-results"></a>
# Adding test coverage results to SonarQube
If you need a refresher on how to use the SonarQube scanner for msbuild, take a look at [my previous post about getting started with SonarQube on a C# project](/2017/07/16/getting-started-with-sonarqube-on-a-csharp-project/).

The general steps we need to take when running a SonarQube scan on a .NET project are the following:
1. Setup the SonarQube scanner for msbuild - using the `begin` argument (retrieve rules from SonarQube instance, inject analyzers in build pipeline)
2. Run msbuild
3. Use the SonarQube scanner for msbuild to generate and submit the report - using the `end` argument

This would give us the following corresponding commands (replace the variables with your parameters):

1. 
```
SonarQube.Scanner.MSBuild.exe begin 
/k:{desired_project_name} 
/d:'sonar.host.url={sonarqube_url}' 
/d:'sonar.host.port={sonarqube_port}' 
/d:'sonar.user={sonarqube_user}' 
/d:'sonar.login={sonarqube_logintoken}' 
```

2.
```
msbuild.exe /t:Rebuild {solution_name}.sln
```

3.
```
SonarQube.Scanner.MSBuild.exe end /d:'sonar.login='{sonarqube_logintoken}'
```

Now we know how to generate a code coverage result file, we can make SonarQube aware of where to pick up the results.

We can do this by modifying step 1 and specifying the `sonar.cs.opencover.reportsPaths` property in our SonarQube scanner command like this:
```
/d:'sonar.cs.opencover.reportsPaths=opencover.xml'
```

SonarQube will pick up our code coverage and show it in our projects dashboard:

![2017-11-05-11_09_47-SonarQubeNUnitIntegrationDemo](/images/2017/11/2017-11-05-11_09_47-SonarQubeNUnitIntegrationDemo.png)