# Setting Up Prometheus Observability Stack Using Docker
![Untitled Diagram drawio](https://github.com/nageshwar50/prometheus-observability/assets/128671109/44a93665-06c8-4da7-9b6e-ac860ff3792b)


This blog covers the step-by-step guide to set up an Observability Stack that includes Prometheus, Grafana, and Alert Manager using Terraform, Docker, and Docker Compose.
## DevOps Tools / Service Used
In this setup, we have used the following open-source DevOps tools.

Docker
Prometheus
Alert Manager
Grafana
Prometheus Node Exporter
Terraform
Following are the AWS services used.

ec2
Following are the Linux concepts covered as part of the setup

Makefile: Used to modify the server IP address in prometheus config.
Linux SWAP: To add swap to the ec2 instance.
Shell Scripts: To install Docker, Docker compose and add swap as part of ec2 system startup.

## Prerequisites
To deploy the Prometheus stack on Docker, we have the following prerequisites:

AWS Account with a key pair.
AWS CLI configured with the account.
Terraform is installed on your local machine.

In our setup, we will be using the following components

## 1. Prometheus
Prometheus is used to scrape the data metrics, pulled from exporters like node exporters. It used to provide metrics to Grafana. It also has a TSDB(Time series database) for storing the metrics

## Alert Manager
Alert manager is a component of Prometheus that helps us to configure alerts based on rules using a config file. We can notify the alerts using any medium like email, slack, chats, etc.

With a nice dashboard where all alerts can be seen from the Prometheus dashboard.

## 3. Node Exporter
Node exporter is the Prometheus agent that fetches the node metrics and makes it available for Prometheus to fetch from /metrics endpoint. So basically node exporter collects all the server-level metrics such as CPU, memory, etc.
While other custom agents are still available and can be used for pushing metrics to Prometheus

## 4. Grafana
Grafana is a Data visualization tool that fetches the metrics from Prometheus and displays them as colorful and useful dashboards.
It can be integrated with almost all available tools in the market.

To deploy the Prometheus stack, we will be using the following DevOps Tools

## 1. Terraform
Terraform is one of the most popular Infrastructures as a Code created by HashiCorp. It allows developers to provision the entire infrastructure by code. 
We will use Terraform to provision the ec2 instance required for the setup.

## 2. Docker
Docker is a tool for packaging, deploying, and running applications in lightweight. 
We will deploy Prometheus components and Grafana on Docker containers.

## 3. Docker Compose
A Docker-based utility to run multi-container Docker applications. It allows you to define and configure the application’s services, networks, and volumes in a simple, human-readable YAML file

# Project IaC Code Explained
All the IaC codes and configs used in this setup are hosted on the DevOps Projects Github repository.
Clone the DevOps projects repository to your workstation to follow the guide.
<pre>
  https://github.com/nageshwar50/prometheus-observability.git
  cd prometheus-observability
  </pre>
<pre>
Note: Use visual studio code or relevant IDE to understand the code structure better.
  </pre>

![image](https://github.com/nageshwar50/prometheus-observability/assets/128671109/056962f6-15ab-460e-ba29-f966edae0e26)

## Let’s understand the project files.

The alertmanager folder contains the alertmanager.yml file which is the configuration file. If you have details of the email, slack, etc. we can update accordingly.

The prometheus folder contains alertrules.yml which is responsible for the alerts to be triggered from the prometheus to the alert manager. The prometheus.yml config is also mapped to the alert manager endpoint to fetch, Service discovery is used with the help of a file `file_sd_configs` to scrape the metrics using the targets.json file.

terraform-aws directory allows you to manage and isolate resources effectively. modules contain the reusable terraform code. These contain the Terraform configuration files (main.tf, outputs.tf, variables.tf) for the respective modules.

The ec2 module also includes user-data.sh script to bootstrap the EC2 instance with Docker and Docker Compose. The security group module will create all the inbound & outbound rules required.

Prometheus-stack Contains the configuration file main.tf required for running the terraform. Vars contains an ec2.tfvars file which contains variable values specific to all the files for the terraform project.

The Makefile is used to update the provisioned AWS EC2’s public IP address within the configuration files of prometheus.yml and targets.json located in the prometheus directory.

The docker-compose.yml file incorporates various services Prometheus, Grafana, Node exporter & Alert Manager. These services are mapped with a network named ‘monitor‘ and have an ‘always‘ restart flag as well.

## Docker Images
We are using the following latest official Docker images available from the Docker Hub Registry.

prom/prometheus
grafana/grafana
prom/node-exporter
prom/alertmanager

Now that we have learned about the tools and tech and IaC involved in the setup, lets get started with the hands-on installation.

## Provision Server Using Terraform

Modify the values of ec2.tfvars file present in the terraform-aws/vars folder. You need to replace the values highlighted in bold with values relevant to your AWS account & region.

If you are using ca-central-1, you can continue with the same AMI id.

<pre>
  # EC2 Instance Variables
region         = "ca-central-1"
ami_id         = "ami-0fdd2bb78451114e8"
instance_type  = "t2.micro"
key_name       = "pandey"
instance_count = 1
volume-size = 20

# VPC id
vpc_id  = "vpc-0cac018cd52b8d4c1"
subnet_ids     = ["subnet-0974592fab983deaf"]

# Ec2 Tags
name        = "prometheus-stack"
environment = "dev"
application = "monitoring"
</pre>

Now we can provision the AWS EC2 & Security group using Terraform.
First, we need to install Terraform
![image](https://github.com/nageshwar50/prometheus-observability/assets/128671109/ee2fa5d6-9316-4544-a901-088b8a3450c6)

<pre>
cd terraform-aws/prometheus-stack/
terraform fmt
terraform init
terraform validate
</pre>

Execute the plan and apply the changes.
<pre>
  terraform plan --var-file=../vars/ec2.tfvars
terraform apply --var-file=../vars/ec2.tfvars
</pre>



































