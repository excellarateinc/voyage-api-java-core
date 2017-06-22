# Overview
A foundational set of web services that implement industry standard guidelines, common best practices, and the experienced insights afforded to Lighthouse Software thru decades of enterprise business software development. 

If you are extending this API to build a new app, then replace this section with a detailed overview of the new app. Include as much or as little detail as necessary to convey to the developers what this project is about. Often times less is more. 

## Topics
* [Features](#features)
* [Development](readme_docs/DEVELOPMENT.md)
* [Development Recipes](readme_docs/DEVELOPMENT-RECIPES.md)
* [Testing](readme_docs/STANDARDS-TESTING.md)

## Features
* __Exception Handling__
  - Basic strucnture of exceptions and I18n standards.
* __Email__
  - Email Handler with model structure 
* __SMS__
  - SMS Handler with model structure 
  
### Tech Stack
* __Spring Boot__
  - Groovy
* __Integrated Test Suite__
  - Automated test coverage using Spock testing framework
  - Tests executed during every build to ensure high quality code coverage

### Developers
* __Team Protocols__
  - Fast learning curve through clear documentation
  - Easy values, standards, best practices that most developers will agree to follow
* __Core Values__
  - Documented core values that we believe will resonate with most development teams
  - Unifies teams and promotes healthy communication
  - See our [Core Values](readme_docs/DEVELOPMENT.md#core-values) documentation
* __Coding Standards__ 
  - Industry accepted language coding standards
  - Best practices when developing within the code base
  - Standard enforced using static code analysis at build time (CodeNarc)
  - See our [Development Team Standards](#development-team-standards)
  
### System Administrators
* __Deploy Instructions__
  - Full instructions on how to properly build, test, and package the API app for deploy
  - Continuous Integration job templates for QA, UAT, and PROD
* __Docker Support__
  - Preconfigured Dockerfile for deployment within Amazon Web Services environment
  - Generate a Docker bundle for distribution using built-in tasks
  - Customize to fit any environment
* __Amazon Web Services (AWS) - Elastic Beanstalk__
  - Supports AWS Elastic Beanstalk using a Docker image
  - Run a build task to generate an AWS EB compatible .zip file
* __DevOps Ready__
  - [Ansible](https://www.ansible.com) scripts for deploying the API Docker image to the Amazon Web Service (AWS) environment
  - Customize scripts to support any environment