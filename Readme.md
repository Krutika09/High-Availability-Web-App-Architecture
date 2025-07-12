# VPC and Auto Scaling Group Setup Guide

## 1. Select VPC and more And resources 

* **Availability Zones**: 2
* **Public Subnets**: 2
* **Private Subnets**: 2
* **NAT Gateways**: 1 per AZ
* **VPC Endpoint**: None

![alt text](image-2.png)

## 2. Create Launch Template

* **Name**: `project-vpc-temp`
* **Auto Scaling Guidance**: Checked

### Launch Template Contents

* **AMI**: Ubuntu
* **Instance Type**: `t2.micro`
* **Key Pair**: Create new key pair (used throughout the project)

![alt text](image-3.png)


### Network Settings

** Only add SG and VPC do not touch Subnets & AZs **
* **Security Group**: Create new SG (Name it appropriately)
* **VPC**: Select created VPC
* **Inbound Rules**:

  * SSH (Port 22)
  * Custom TCP (Port 8000)

![alt text](image-14.png)

## 3. Create Auto Scaling Group

* **Name**: (Give a name)
* **Launch Template**: Select `project-vpc-temp`
* **VPC**: Select the created VPC
* **Subnets**: Select both **Private Subnets**
* **Integration**: Default

### Configure Group Size and Scaling

* **Desired Capacity**: 2
* **Min**: 2
* **Max**: 4

![alt text](image-4.png)

## 4. Create Bastion Host (Instance)

* **AMI**: Ubuntu
* **Instance Type**: `t2.micro`
* **VPC**: Select the created VPC
* **Subnet**: Select **Public Subnet**
* **Auto-assign IP**: Enabled
* **Security Group**: Select the same SG used earlier

![alt text](image-5.png)

## 5. Connect Using Git Bash Terminal

```bash
cd Downloads
scp -i /c/Users/Krutika/Downloads/vpc-project.pem /c/Users/Krutika/Downloads/vpc-project.pem ubuntu@<Bastion_Public_IP>:/home/ubuntu
```

![alt text](image-6.png)


```bash
ssh -i /c/Users/Krutika/Downloads/vpc-project.pem ubuntu@<Bastion_Public_IP>
```

![alt text](image-7.png)

### Inside Bastion Host

```bash
ls  # Check PEM file presence
chmod 400 vpc-project.pem
sudo ssh -i vpc-project.pem ubuntu@<Private_Instance_IP>
```

![alt text](image-8.png)

### Add Web Content

```bash
echo "<h1>Welcome to Instance 1</h1>" > index.html
python3 -m http.server 8000
```

![alt text](image-9.png)

## 6. Create Load Balancer

### Target Group

* **Target Type**: Instance
* **Name**: `project-vpc-tg`
* **Protocol**: HTTP
* **Port**: 8000
* **VPC**: Select self-created VPC
* **Protocol Version**: HTTP1
* **Register Targets**: Select those 2 instances → Click *Include as pending below*

![alt text](image-10.png)
![alt text](image-11.png)

### Application Load Balancer (ALB)

* **Name**: `project-vpc-alb`
* **Scheme**: Internet Facing
* **VPC**: Select self-created VPC
* **Subnets**: Select **Public Subnets**

![alt text](image-12.png)

### Listeners & Routing

* **Listener 1**: HTTP, Port 8000 → Forward to `project-vpc-tg`
* **Listener 2**: HTTP, Port 80 → Forward to `project-vpc-tg`

![alt text](image-13.png)

### Access DNS from browser

```
http://project-vpc-alb-184731419.us-west-2.elb.amazonaws.com/
```

![alt text](image-15.png)

![alt text](image-16.png)