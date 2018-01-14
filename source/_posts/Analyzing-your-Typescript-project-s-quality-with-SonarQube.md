---
title: Analyzing your Typescript project's quality with SonarQube
tags:
  - SonarQube
  - Code quality
  - Typescript
categories:
  - Software development
permalink: analyzing-your-typescript-projects-quality-with-sonarqube
id: 59d3e06faf4f1298d16e0ec8
updated: '2017-11-06 07:37:38'
date: 2017-09-27 19:35:49
---
# Table of contents
* [Intro](#intro)
* [Installing the plugins](#plugins)
* [The SonarQube scanner](#scanner)

#  <a name="intro"></a> Intro
After my previous SonarQube blogpost for C# projects, I wanted to figure out if SonarQube could work for a Typescript application. Turns out, it does!

# <a name="plugins"></a>Installing the plugins
To be able to scan our Typescript project with Sass styling, we need to install the following plugins: 
- SonarTS (for the typescript support). This plugin will run tslint on your project.
- SCSS, LESS, CSS plugin (for our styling). This plugin will scan our styling files. It can find issues such as non standard attribute values, non-conventional names, unknown pseudo-selectors and empty rules.
- Web plugin (for our Html): this plugin finds issues such as: accessibility issues (for example using em tags instead of i tags), missing end tags, missing required attributes, ...

You can install them by navigating to "Administration > System > Update Center > Available".
# <a name="scanner"></a>The SonarQube scanner
You can download the SonarQube scanner [here](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner). (caution: this is not the same scanner as the SonarQube msbuild scanner from the previous posts)

We can scan our Typescript project like this:
```
sonar-scanner -D "sonar.projectKey=<projectname>" 
-D "sonar.host.url=<sonar instance url>" -D "sonar.host.port=<sonar instance port>" 
-D "sonar.user=<sonar user>" 
-D "sonar.login=<sonar password / token >" 
-D "sonar.sources=<root directory of your source, relative to working directory>"
-D "sonar.projectVersion=<optional but handy to track progress over versions>"
```

As we can see, our project shows up in the dashboard.

![SonarTs_1](/content/images/2017/09/SonarTs_1.png)