---
title: "Infrastructure as code with Terraform"
date: 2021-04-18T12:53:08+10:00
draft: false

categories: [Platform Engineering]
tags: [infrastructure as code, terraform, aws, devops]
toc: false
author: "Nick Perkins"
---
Over the last couple of months I've been working on a project for a containerised orchestration platform using Infrastructure as Code. This has meant I've spent most of my time working with [Terraform](https://www.terraform.io/).

## What is Terraform?

I could write a couple of paragraphs about what Terraform is and what it does, but I think they put it best:

> Terraform is an open-source infrastructure as code software tool that provides a consistent CLI workflow to manage hundreds of cloud services. Terraform codifies cloud APIs into declarative configuration files.

The best part about Terraform, other than other options such as AWS CloudFormation, is that it is provider-agnostic. In fact, Terraform boast of supporting over 500 different providers within their registry.

If you want to have your DNS in Cloudflare, your server instances in AWS EC2, and store data in Google Cloud Platform Big Data, you can manage all that within a single terraform deployment. You can share information, or outputs, from each service and link them together with a minimum of fuss.

Another great feature is the ability to build configurable and reusable modules. Create a module for setting up your default core network, and then reuse that to deploy that across multiple regions, environments, accounts, and more. You can also set variable blocks to allow configuration of the details of each deployment. You might only use one availability zone for your development environments, but up that to three for production.

## Example Code

I've put together a [repository](https://github.com/nickperkins/infrastructure) which shows just how easy it is to set up a core network with AWS including:

* VPC
* Public and Private Subnets
* An internet gateway for the public subnet
* NAT Gateways for the private subnets
* Routing tables to bring it all together
