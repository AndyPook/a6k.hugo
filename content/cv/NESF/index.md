---
title: NES Fircroft
description: Timesheet System
date: 2023-03-01
jobDate: 2023-Mar - present
job: Contract Developer
role: [code, architecture, devops]
techs: [dotnet, csharp, Kubernetes, postgres, gitlab, github, pwsh]
projectUrl: https://nesfircroft.com
---

Timesheets project:
NESF provides services for Contractors to define time worked and expenses incurred with an approval process. Approved work is passed to another system for invoicing and payment.

Initially, maintenance and support for existing system. Written with nodejs and couchdb, hosted on kubernetes.
Migrated from third-party consultancy, to ownership by NESF. Including moving from gitlab to github.

Created new system from scratch to support requirement for offline mobile app. Many Contractors work in dificult locations with very intermittant/unreliable connectivity. The client apps are designed to be offline first, with messages used to sync actions between backend and other participants.

Created services for integration with payments system. This system also provided definitions of Contractors, Approvers, and contract details (such as start/end dates, rates etc).
Significant portion of this part was in agreeing api surface, and handling throughput limits and resiliency.

Handled several several dotnet version updates. Along with updates required for CVEs. Build and test actions. Publishing container images for each service. Helm charts and assocated parts for deploying product and dependencies to various kubernetes environments.
Collaborated with infrastructure/operations teams on hosting/network behaviour (k8s, DNS, gateway, Ingress etc).
Support for other developers, testers and production systems.

dotnet 8/9/10 with aspnetcore, postgresql.
Hosted on Azure with kubernetes. 
Github actions, helm, operational systems.