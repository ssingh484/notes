# Cyber Workflow automation engine

# Background:

## Problems:
- Repetitive tasks are performed continuously and use multiple people
- Need to develop a common protocol for transferring experience among individuals
- Need a structured approach for accumulating and enhancing knowledge in the org
- Ensuring consistent efficiency
- Setting up similar processes separately for CART, DevOps, DevSecOps and ML is necessary

## Solutions

- BlackCart - open source containerized infra
- Software Factory Methodology - by Lockheed Martin
- YAML Playbooks - Newcomers can learn from YAMLs and declarative automation

## Features
 - DAG based paralellism and CronJob Mechanisms
 - REST API support and small binary allows agile deployment
 - Creating a public YAML collection and scenarios for knowledge transfer
 - WebUI dashboard 
 - 4 languages for DevSecOps playbooks Go, Java, Python, Javascript (PHP, C, C++ in unpublished)
 - Logging and web code editor for the YAML
 - DAG based workflows

## Framework components
- BlackCart - one-click infra creation
- BlackDagger YAMLs - collection
- BlackDagger Web Kit - Browser integration:
	- Various blackdagger tooling, tech stack detection, report generation
- BlackDagger GitHub Infra - Methodology : Use GitHub actions and workflows to conduct attacks (uses Azure infra and these sources have Azure Headers which targets don't tend to block)
## Demo

Using vulnweb.com by Acunetix as "real" target
DAG described by YAML file like a GitActions flow
Live data like in a pipeline status
History page for previous logs
Direct infra term access from the web UI


# More playbooks to come:
- ML SecOps tests