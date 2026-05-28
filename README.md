\# AWS EC2 Web Server with Terraform



\## Project Overview

Deployed a live Apache web server on AWS EC2 using Terraform for Infrastructure as Code (IaC).



\## Tech Stack

\- Terraform

\- AWS EC2 (t3.micro - Free Tier)

\- AWS IAM

\- Apache HTTP Server



\## What This Does

\- Creates an AWS Security Group allowing HTTP traffic on port 80

\- Launches an EC2 instance with Amazon Linux 2

\- Automatically installs and starts Apache via user data script

\- Hosts a live HTML webpage accessible from anywhere



\## How to Deploy

1\. Install Terraform and AWS CLI

2\. Configure AWS credentials: `aws configure`

3\. Clone this repo and cd into it

4\. Run `terraform init`

5\. Run `terraform plan`

6\. Run `terraform apply`



\## How to Destroy

Run `terraform destroy` to remove all AWS resources.

