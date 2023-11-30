# Setting Up Prometheus Observability Stack Using Docker
![Untitled Diagram drawio](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/944bc774-9781-4da0-81ff-743e5b1a9e36)



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

![Screenshot from 2023-11-30 12-56-44](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/66517e8e-4c41-4505-98bf-20a626e53edb)


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

![Screenshot from 2023-11-30 13-03-05](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/cbd699b8-6a47-46cc-a4f5-7101a07273df)


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

Before typing ‘yes‘ make sure the desired resources are being created. 
<pre>
  Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + instance_public_dns = [
      + (known after apply),
    ]
  + instance_public_ip  = [
      + (known after apply),
    ]
  + instance_state      = [
      + (known after apply),
    ]

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: 

</pre>
![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/24148539-9fdf-4de1-a4ad-2e04b09c2221)

Now we can connect to the AWS EC2 machine just created using the public IP. Replace the key path/name and IP accordingly.

<pre>
  ssh -i pandey.pem ubuntu@34.216.95.97

</pre>

We will check the cloud-init logs to see if the user data script has run successfully

<pre>
  tail /var/log/cloud-init-output.log
</pre>
![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/9d1f993c-ee78-4a1e-925a-915e463ae252)

Let’s verify the docker and docker-compose versions again.
<pre>
  sudo docker version
sudo docker-compose version
</pre>

Now that we have the instance ready with the required utilities, let’s deploy the Prometheus stack using docker-compose.

## Deploy Prometheus Stack Using Docker Compose

First, clone the project code repository to the server.
<pre>
  https://github.com/nageshwar50/prometheus-observability-.git
  cd prometheus-observability-/
</pre>

Execute the following make command to update server IP in prometheus config file. Because we are running the node exporter on the same server to fetch the server metrics. We also update the alert manager endpoint to the servers public IP address.

<pre>
  make all
</pre>
![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/9c42c293-9219-430e-8719-486d04a7f30e)

Bring up the stack using Docker Compose. It will deploy Prometheus, Alert manager, Node exporter and Grafana
<pre>
  sudo docker-compose up -d
</pre>
On a successful execution, you should see the following output saying Running 5/5

![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/dd392abf-7cc9-43d9-ae8b-4f67f059a8de)

Now, with your servers IP address you can access all the apps on different ports.

Prometheus: http://your-ip-address:9090
Alert Manager: http://your-ip-address:9093
Grafana: http://your-ip-address:3000

Now the stack deployment is done. The rest of the configuration and testing will be done the using the GUI.

## Validate Prometheus Node Exporter Metrics

If you visit http://your-ip-address:9090, you will be able to access the Prometheus dashboard as shown below.

Validate the targets, rules and configurations as shown below. The target would be Node exporter url.

Status -> Targets
![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/ddf6d4ae-5916-47fd-8122-492e25c7b5c4)

Status -> Rules
![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/f3730ae8-2fc4-4464-95db-c4f0e4b28fe0)

Status -> Configuration
![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/035bae01-d8c4-42bd-a299-2a4c28c0531b)

Now lets execute a promQL statement to view node_cpu_seconds_total metrics scrapped from the node exporter.
<pre>
  avg by (instance,mode) (irate(node_cpu_seconds_total{mode!='idle'}[1m]))
</pre>
You should be able to data in graph as shown below.
home -> add panel
![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/548dca14-5fff-4957-ba7f-02c69fe56091)

## Configure Grafana Dashboards

Now lets configure Grafana dashboards for the Node Exporter metrics.

Grafana can be accessed at: http://your-ip-address:3000

Use admin as username and password to login to Grafana. You can update the password in the next window if required.

Now we need to add prometheus URL as the data source from Connections→ Add new connection→ Prometheus → Add new data source.

Here is the demo.

![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/425d4ea0-6491-4d42-9371-52eb39bcc8ab)

![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/c947beef-066b-4dd3-941f-f56becb825f6)

## Configure Node Exporter Dashboard
Grafana has many node exporter pre-built templates that will give us a ready to use dashboard for the key node exporter metrics.

To import a dashboard, go to Dashboards –> Create Dashboard –> Import Dashboard –> Type 10180 and click load –> Select Prometheus Data source –> Import

![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/2f84211a-6952-4acf-bd1f-201709ff6a92)

Once the dashbaord template is imported, you should be able to see all the node exporter metrics as shown below.

![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/3e5265b0-68f1-4f53-8b3a-8c7fbec63723)

## Simulate & Test Alert Manager Alerts
You can access the Alertmanager dashbaord on http://your-ip-address:9093

![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/fec7784d-838a-49f7-b814-bfad8e392e0c)

Alert rules are already backed in to the prometheus configuration through alertrules.yaml. If you go the alerts option in the prometheus menu, you will be able to see the configured alerts as shown below.

![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/681319f3-5960-4c95-8873-ca45e0944c2c)

As you can see, all the alerts are in inactive stage. To test the alerts, we need to simulate these alerts using few linux utilities.

You can also check the alert rules using the native promtool prometheus CLI. We need to run promtool command from inside the prometheus container as shown below.

<pre>
  sudo docker exec -it prometheus promtool check rules /etc/prometheus/alertrules.yml
</pre>

## Test: High Storage & CPU Alert

<pre>
  dd if=/dev/zero of=testfile_16GB bs=1M count=16384; openssl speed -multi $(nproc --all) &
</pre>

![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/541fdd39-1594-4db4-8c00-708862e0f575)

Now we can check the Alert manager UI to confirm the fired alerts. 

![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/ac08f442-3a21-4427-ac32-8f14149c26ac)

Now let’s rollback the changes and see the fired alerts has been resolved.
<pre>
  rm testfile_16GB && kill $(pgrep openssl)
</pre>

![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/7dbe48ed-2080-4e51-9bf2-7cf4929c4933)

![image](https://github.com/nageshwar50/prometheus-observability-/assets/128671109/e5532ccd-b63a-4c00-81c7-dc90993a8061)

# Cleanup The Setup
To tear down the setup, execute the following terraform command from your workstation.

terraform destroy --var-file=../vars/ec2.tfvars

## Possible Errors
If you don’t have the correct ec2 AMI id set in the vars file, you will get the following error. To rectify the issue, update the correct AMI ID related to the region you are using.

<pre>
  Error: creating EC2 Instance: InvalidAMIID.NotFound: The image id '[ami-06873c81b882339ac]' does not exist
│       status code: 400, request id: 34d38d4d-c3b8-47e6-9c27-1b1cbccbab83
</pre>


## Conclusion
As a quick recap we learned to provision the AWS infra using Terraform.

Then we brough up the Prometheus Observability stack using Docker compose and configured Grafana to create Node Exporter metrics dashboard.

Also we simulated alerts to check the validate the alerting.

If you’re facing any kind of issues, let me know in the comment section.
























