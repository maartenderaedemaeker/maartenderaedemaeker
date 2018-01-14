---
title: Using OWASP Dependency check with SonarQube
tags:
  - SonarQube
  - C#
  - Security Testing
  - Java
categories:
  - Software development
permalink: using-owasp-dependency-check
id: 59d3e06faf4f1298d16e0ec4
updated: '2017-08-13 22:40:10'
date: 2017-07-27 20:41:42
---
# Intro
In this post I will discuss how to use OWASP Dependency check to help mitigate the security risk of using component with known vulnerabilities.
<a href="https://github.com/maartenderaedemaeker/Automated-SecurityTesting-Demo/tree/part2">Demo code used for scanning: see github.</a>
# Table of contents
* <a href="#owaspTopTen">OWASP top 10</a>
* <a href="#knownvulnerabilities">Using components with known vulnerabilities</a>
* <a href="#owaspdependencycheck">OWASP Dependency check</a>
* <a href="#usingdependencycheck">Using OWASP dependency check</a>
* <a href="#dependencychecksonar">Getting OWASP dependency check reports in SonarQube</a>
* <a href="#conclusion">Conclusion</a>

# <a name="owaspTopTen"></a> OWASP top 10
If you haven't heard about OWASP yet, their name is short for "Open Web Application Security Project". It's an organization trying to improve Web application security. They are very known for their "top 10" project, which they release every few years.
It is a list with the 10 highly rated security related risks.
They use 4 criteria to rate the risks: 
- Attack vectors (how easy is it to exploit this risk?)
- Weakness Prevalence (how often does this risk occur)
- Weakness detectability (is it easy to find the risk?)
- Technical impact (what could happen when the risk is exploited?)

For more information: visit <a href="https://www.owasp.org/index.php/Main_Page" target="_blank">OWASP</a>.

# <a name="knownvulnerabilities"></a>Using components with known vulnerabilities
One of the risks in the top 10 is using components with known vulnerabilities.
This boils down on being dependent on libraries / frameworks / tools with known security risks.
These vulnerabilities are often called CVE's (Common Known Vulnerabilities). You can look them up on sites such as <a href="https://www.cvedetails.com/">these</a>.

# <a name="owaspdependencycheck"></a> OWASP Dependency check
OWASP Dependency check is a project run by OWASP that allows us to scan .NET and Java projects. The tool also has experimental support for a few other languages.
<a href="https://www.owasp.org/index.php/OWASP_Dependency_Check">You can get more info here.</a>

# <a name="usingdependencycheck"></a> Using OWASP dependency check
So, let's get our hands dirty. First of all, you can download the commandline version <a href="https://www.owasp.org/index.php/OWASP_Dependency_Check"> here </a>.
![Downloading_owasp_dependencycheck](/images/2017/07/27/Downloading_owasp_dependencycheck.png)

After downloading, I extracted the tool to C:\tools\dependency-check and added the bin folder to my path variable.

Ok. Now I can use the tool from cmd or powershell window. First of all the tool will start downloading definitions for known vulnerabilities (which takes a while the first time, after that, only updates are retrieved).

```
dependency-check --scan <folder to scan> --project <project name used in report>
```

![Downloading_owasp_dependencycheck-1](/images/2017/07/27/Downloading_owasp_dependencycheck-1.png)

After updating, the scan will start. When the scan is done, you'll notice the output (a HTML file). When opening the file, you'll see an overview of possible issues and links to why they may be an issue. ![DependencyCheckResults](/images/2017/07/27/DependencyCheckResults.png).

Great, we now know how to get started! :-)

# <a name="dependencychecksonar"></a> Getting OWASP dependency check reports in SonarQube
We can import our OWASP Dependency check reports in SonarQube by using following <a href="https://github.com/stevespringett/dependency-check-sonar-plugin">plugin.</a>

To install it, copy the downloaded jar file to <your sonar installation path>\extensions\plugins and restart the SonarQube service.

You should now see the plugin in the SonarQube update center.
![SonarInstalled](/images/2017/07/27/SonarInstalled.png)

We can use the report created by OWASP Dependency check if we make sure we choose the xml format instead of the default html. (see below)

```
dependency-check --scan <folder to scan> --project <project name used in report> -f XML
```

Now, if we specify the following flag while executing our Sonar Scanner, the generated xml file will be submitted as well.
```
 /d:"sonar.dependencyCheck.reportPath=<FILL_PATH_TO_REPORT_XML_IN_HERE>"
```

I had some issues to get this setup at first. The latest binary download was not compatible with my 6.X SonarQube instance, but cloning the git repository and building it locally worked for me. I also needed to specify the full path to the report to make it work.

When running the scan, you should see something like this in your output:
![DepCheck](/images/2017/07/27/DepCheck.png)

When looking at the issues on your SonarQube, you should notice them showing up as vulnerabilities.

![DependencyIssues](/images/2017/07/27/DependencyIssues.PNG)

# <a name="conclusion"></a> Conclusion
I really like this "low-effort way" of creating visibility for the "Using components with known vulnerabilities" - risk and keeping track of the issues they bring in C# / Java code. I will take some time later on to figure out and create a blog about finding these issues on NodeJS / Front end Javascript, for a more complete approach.