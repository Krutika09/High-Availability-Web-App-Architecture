# VPC and Auto Scaling Group Setup Guide

## 1. Select VPC and more And resources 

* **Availability Zones**: 2
* **Public Subnets**: 2
* **Private Subnets**: 2
* **NAT Gateways**: 1 per AZ
* **VPC Endpoint**: None

<img width="1332" height="449" alt="image" src="https://github.com/user-attachments/assets/24894c68-5dd4-40bd-b377-5da3f26c42c7" />


## 2. Create Launch Template

* **Name**: `project-vpc-temp`
* **Auto Scaling Guidance**: Checked

### Launch Template Contents

* **AMI**: Ubuntu
* **Instance Type**: `t2.micro`
* **Key Pair**: Create new key pair (used throughout the project)

<img width="1136" height="162" alt="image" src="https://github.com/user-attachments/assets/4c0ea2c5-122e-4309-9715-becd70722aa9" />


### Network Settings

** Only add SG and VPC do not touch Subnets & AZs **
* **Security Group**: Create new SG (Name it appropriately)
* **VPC**: Select created VPC
* **Inbound Rules**:

  * SSH (Port 22)
  * Custom TCP (Port 8000)

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

<img width="1105" height="196" alt="image" src="https://github.com/user-attachments/assets/06813d44-3e7b-41cb-b0f9-58ae781e6970" />

## 4. Create Bastion Host (Instance)

* **AMI**: Ubuntu
* **Instance Type**: `t2.micro`
* **VPC**: Select the created VPC
* **Subnet**: Select **Public Subnet**
* **Auto-assign IP**: Enabled
* **Security Group**: Select the same SG used earlier

<img width="1109" height="195" alt="image" src="https://github.com/user-attachments/assets/40014f8b-f100-420e-a4c8-8d86bf98d06c" />

## 5. Connect Using Git Bash Terminal

```bash
cd Downloads
scp -i /c/Users/Krutika/Downloads/vpc-project.pem /c/Users/Krutika/Downloads/vpc-project.pem ubuntu@<Bastion_Public_IP>:/home/ubuntu
```

<img width="752" height="319" alt="image" src="https://github.com/user-attachments/assets/0e174704-27cf-4943-9dad-a8fd4bc78011" />


```bash
ssh -i /c/Users/Krutika/Downloads/vpc-project.pem ubuntu@<Bastion_Public_IP>
```

<img width="601" height="536" alt="image" src="https://github.com/user-attachments/assets/b6b9e141-2785-4cfb-b000-7cd1388f0130" />

### Inside Bastion Host

```bash
ls  # Check PEM file presence
chmod 400 vpc-project.pem
sudo ssh -i vpc-project.pem ubuntu@<Private_Instance_IP>
```

<img width="627" height="663" alt="image" src="https://github.com/user-attachments/assets/e54677c8-3e65-42bb-a96d-282b248da7a1" />

### Add Web Content

```bash
echo "<h1>Welcome to Instance 1</h1>" > index.html
python3 -m http.server 8000
```

<img width="695" height="465" alt="image" src="https://github.com/user-attachments/assets/b4abdab5-a677-4390-9d7c-8d8e08488965" />

## 6. Create Load Balancer

### Target Group

* **Target Type**: Instance
* **Name**: `project-vpc-tg`
* **Protocol**: HTTP
* **Port**: 8000
* **VPC**: Select self-created VPC
* **Protocol Version**: HTTP1
* **Register Targets**: Select those 2 instances → Click *Include as pending below*

<img width="1116" height="407" alt="image" src="https://github.com/user-attachments/assets/e87df5c7-90aa-4817-93a8-4835ac1bf026" />
<img width="1109" height="319" alt="image" src="https://github.com/user-attachments/assets/3f02820f-959f-47e1-bfd8-d92ebff4184c" />

### Application Load Balancer (ALB)

* **Name**: `project-vpc-alb`
* **Scheme**: Internet Facing
* **VPC**: Select self-created VPC
* **Subnets**: Select **Public Subnets**

<img width="1105" height="349" alt="image" src="https://github.com/user-attachments/assets/f9c2c7fc-c261-4b5c-8c4f-d950ee9511c2" />

### Listeners & Routing

* **Listener 1**: HTTP, Port 8000 → Forward to `project-vpc-tg`
* **Listener 2**: HTTP, Port 80 → Forward to `project-vpc-tg`

<img width="1105" height="365" alt="image" src="https://github.com/user-attachments/assets/7d791d0c-3036-422b-8ddd-b59ef204c337" />
<img width="1143" height="490" alt="image" src="https://github.com/user-attachments/assets/d99b08e2-19a0-486f-b42f-effea2a4f537" />


### Access DNS from browser

```
http://project-vpc-alb-184731419.us-west-2.elb.amazonaws.com/
```

<img width="956" height="471" alt="image" src="https://github.com/user-attachments/assets/d6f5b300-86ab-418c-b4a8-c92dd9c8f5ed" />

<img width="1175" height="473" alt="image" src="https://github.com/user-attachments/assets/f1a2fb2d-cf5f-4854-9e4f-7b9e8643ab99" />
