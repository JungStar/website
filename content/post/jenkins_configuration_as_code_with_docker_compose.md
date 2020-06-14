---
title: "Jenkins Configuration as Code with Docker Compose"
summary: "How to setup a Jenkins server with Docker Compose and Jenkins Configuration as Code (JCasC)"
draft: false
date: "2020-06-12T00:00:00Z"
authors: 
  - philippjung
tags:
  - Jenkins
  - Docker
  - Docker-Compose
  - Infrastructure as code
---
## Why Configuration as Code?
I have been using Jenkins heavily for automation and CI/CD tasks over the last year. Pretty fast I got excited and started automating as many of my regular tasks as possible. And Jenkins is a great tool for that. However, soon I realized that the state of my Jenkins server was hard to reproduce, and should my infrastructure ever fail, there wouldn't be an easy way to get back to my current configuration. Sure, I had a regular backup of the Jenkins configuration (i.e., Jenkins home). But those are mainly XML files that are hard to read and almost impossible to maintain. The solution: Use declarative pipelines, i.e., Jenkinsfiles, that are tracked by git (or any other SCM). This way at least the jobs themselves are backed up. But what about the rest of the Jenkins configuration: Plugins, credentials or libraries. As of 2018 there exist Jenkins Configuration as Code (JCasC), which uses easy to understand `yaml` files to capture the Jenkins configuration. In the following we will see how such a basic configuration could look like. And as a little bonus, we can even script the "seed" jobs that use Jenkinsfile from SCM checkouts using the `job-dsl` plugin

