# RFD 1 Shieldbattery Test Infrastructure

\- \
**authors**: William Hammond <william.t.hammond@gmail.com> \
**state**: discussion \
\-

## Overview

ShieldBattery is an open source, community-run server, and client for playing Starcraft Broodwar. 
This RFD proposes a solution for repeatable infrastructure tests of Shield battery.

## Problem

 Local testing is primarily driven by using a Docker setup. 
 This is fine for quickly iterating locally, but it limits the ability to do controlled acceptance testing. 
 Additionally, ShieldBattery continues to add more features than the Blizzard client, leading to a quickly increasing player count. 
 Having the ability to spin up test infrastructure easily will allow for improved performance testing. 

## Implementation

### Principles

Guiding the implementation of this system are the following principles.
* Reproducibility. This system should be easy to set up and easy to tear down. 
* Thrift. ShieldBattery is an open-source, community-funded project. Resources need to be put to use as effectively as possible.
* Security and reliability. We're building this on real infrastructure, so it needs to be isolated from production. 
* Programmability. Spinning up and tearing down this infrastructure should be automated to let us use it for computer-driven tests.

### Overview

At a high level, we propose using an infrastructure-as-code (IaC) to handle defining and standing up the infrastructure. 
This will allow us to ensure reproducibility and keep the solution open-source. 
The IaC solution will need a solid integration with our CI/CD solution of choice, which in our case is Github Actions. 
We don't want to over-abstract, but we want to avoid vendor lock-in as much as possible so we can chase hosting deals. 

### IaC

Terraform is an IaC tool created by Hashicorp that uses a custom template language to define infrastructure. 
A special state file tracks the state of the infrastructure. 

The templating language makes configuration simple; we could reuse the code for standing up infrastructure for automated tests and as a playground for individual interactive testing. 

Terraform has an officially supported Github Action https://github.com/hashicorp/setup-terraform we can use to automate the creation of infrastructure for acceptance tests. 

The primary issue with using Terraform for this is the state file. 
We don't really care about the state since the infrastructure is being thrown away, but at the same time, we don't want to be too haphazard with managing the state file since it could contain secrets. 
We're already using S3, and it's a supported backend, so it probably makes the most sense to just use that, assuming we can't find a secure way to keep the state file on the Github Action build machines. 

### Security

Throwaway infrastructure is still real infrastructure. 
If we aren't careful with our credentials, our infrastructure could be misused, and we'll have to pay for it. 
State files hold unencrypted, sensitive data like database passwords https://www.terraform.io/language/state/sensitive-data. 

We could set up something like Hashicorp's Vault to manage credentials, but that seems like overkill for a project like this. 
Github actions support cron-like scheduling that we could use to generate new randomized credentials daily.


The open-source IaC code won't know about our infrastructure, so other users can tweak it or provide their own credentials without risk to our systems. 

### Automation

Hashicorp has an officially supported GitHub Action we could use for automated testing https://github.com/hashicorp/setup-terraform. 

## Open Issues 

* Do we need to keep the credential generation script private? 
My knowledge of random number generators is limited, so I'm unsure if there is a way to keep it open-source.
* Is there a secure way to leave the state file on the build machines?
* How should we decide when to spin up the test infrastructure automatically? 
It'll definitely be overkill for all merges to main. 
Should we do it based on critical code paths, or just let the start be manual?
