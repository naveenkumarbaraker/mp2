# Data Clustering using Firefly Algorithm in Distributed Environment

## 1 – Creating the t2.micro Instances
Log into the AWS Management Console at https://console.aws.amazon.com/. On the Console Home page, click the Services button at the top to access the pull-down menu.

Select Services -> Compute -> EC2 to open the EC2 Dashboard. From here, you can find lists of resources and a button for launching an instance.

In the Name and tags section on the Launch an instance page, name the new instance "Worker".
<p align="center">
  <img src="https://gateway.pinata.cloud/ipfs/QmRvT2XRKgNGFQq24krGao1W9HoBrn48coDJG6dsvXfuV5" width="500" >
</p>

In the Application and OS Images section, choose an Amazon Machine Image (AMI). In the Instance type section, select the default t2.micro instance, which is free tier eligible.

In the Key pair (login) section, click the Create new key pair button to specify a key pair for SSH communication. Amazon EC2 stores the public key, and you must store the private key securely.
Click Create new key pair, name it hadoopkey, select RSA type and .ppk format, then click Create key pair. The hadoopkey.ppk file will download automatically.

Use the default settings in Network settings and Configure storage. In the Summary section, enter 4 for the Number of instances.

Click Launch instance to create four instances. A confirmation will show their successful creation.

## 2 – Configure Security Group
### 2.1 Access the Inbound Rules: 
Select the master node in the instance list to view configurations. This allows access to modify the security group settings, ensuring all instances share the same rules.

### 2.2 Open Port 9870 to the Web Interface:
Clic Edit inbound rules, then Add rule to open port 9870 for the web interface. 

### 2.3 Open All Ports Only for the Same Security Group:
Add a new inbound rule allowing ports 0-64000 for the same security group as configured. This ensures comprehensive cluster security without exposing unnecessary access points.
Click on the Save Rules button to accept all the changes.

## 3 – Installing Hadoop on the Master Node
### 3.1 The Master Node Connection
Choose Instances in the navigation pane, select the master instance from the list, and click Connect. On the Connect to instance page, choose Connect using EC2 instance Connect and click Connect to open the terminal window.

### 3.2 Java Installation
Hadoop requires Java Runtime Environment (JRE) 1.6 or higher . We run these two commands to install Java on the master node.
```
sudo yum install java-1.8.0
sudo yum install java-1.8.0-devel
```

### 3.3 Hadoop Installation
Deploying Hadoop on a multi-node cluster starts with downloading and installing Hadoop on the master instance, followed by configuring it for your setup and requirements.

#### 3.4.1 Hadoop 3.3.6 Installation
We run the following command to download the Hadoop 3.3.6 installation package to the master node
```
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
```

Then, we use the following command to unpack the installation package






