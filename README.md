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
```
$ tar xzf hadoop-3.3.6.tar.gz
```
### 3.4.2 JAVA_HOME Configuration

Next, we use the readlink command to determine the location of the Java installation:
```
$ readlink -f $(which java)
```
The command returns the following path:
```
/usr/lib/jvm/java-1.8.0-amazon-corretto.x86_64/jre/bin/java
```
We remove the string "bin/java" from the path and use the modified path as the JAVA_HOME configuration:
```
/usr/lib/jvm/java-1.8.0-amazon-corretto.x86_64/jre
```
With the path, we modify the conf/hadoop-env.sh to set the JAVA_HOME configuration. We first use the nano text editor to open the hadoop-env.sh file:
```
$ nano hadoop-3.3.6/etc/hadoop/hadoop-env.sh
```
After finding the line "# export JAVA_HOME=" in the editor, we add the configuration line below the original line, as shown in Figure 21. We can hit Ctrl + O, then the Entry key to save the changes. Next, we hit Ctrl + x to exit the editor.

Figure 21 Set the JAVA_HOME configuration in the conf/hadoop-env.sh file.
Figure 21 Set the JAVA_HOME configuration in the conf/hadoop-env.sh file.

### 3.4.3 Environment Settings

We use the nano ~/.bashrc command to modify the .bashrc file. We add these two lines:
```
export HADOOP_HOME=~/hadoop-3.3.6
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```
### 3.4.3 Environment Settings

We use the nano ~/.bashrc command to modify the .bashrc file. We add these two lines:
```
export HADOOP_HOME=~/hadoop-3.3.6
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

After saving the changes and exiting the text editor, we run the following command to refresh the settings:
```
$ source ~/.bashrc
```
Next, we find the IP address of each node from the Networking tab shown in Figure 13. Here are four Private IP addresses used in this exercise:

Master: 172.31.35.150
Worker 1: 172.31.34.82
Worker 2: 172.31.33.70
Worker 3: 172.31.34.132
We use these IP addresses to build some configuration files. The IP addresses should differ from this exercise whenever we create a new cluster. Therefore, we should change the IP addresses in the configuration files accordingly.

### 3.4.4 The core-site.xml File

Run the following command to open the core-site.xml file:
```
% nano hadoop-3.3.6/etc/hadoop/core-site.xml
```
We add the following XML elements into the file. Note that the IP address is the private IP address of the master node (Malik, 2023). The file in the text editor should look like Figure 23.
```
<configuration>
    <property>
         <name>fs.defaultFS</name>
         <value>hdfs://172.31.35.150/</value>
    </property>
</configuration>
```

### 3.4.5 The hdfs-site.xml File

We use the following command to open the hdfs-site.xml file:
```
$ nano hadoop-3.3.6/etc/hadoop/hdfs-site.xml
```

We add the following settings to the XML file:
```
<configuration>
   <property>
    <name>dfs.replication</name>
    <value>2</value>
   </property>
   <property>
     <name>dfs.name.dir</name>
     <value>file:///home/ec2-user/dfs/name</value>
   </property>
   <property>
     <name>dfs.data.dir</name>
     <value>file:///home/ec2-user/dfs/data</value>
   </property>
</configuration>
```
### 3.4.6 The mapred-site File

Open the mapred-site.xml file in the nano text editor:
```
$ nano hadoop-3.3.6/etc/hadoop/mapred-site.xml
```
We add the following settings to the XML file (Holmes, 2016):
```
<configuration>
   <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
   </property>
   <property>
      <name>mapreduce.map.memory.mb</name>
      <value>2048</value>
   </property>
   <property>
      <name>mapreduce.reduce.memory.mb</name>
      <value>4096</value>
   </property>
   <property>
      <name>mapreduce.map.java.opts</name>
      <value>-Xmx1638m</value>
   </property>
   <property>
      <name>mapreduce.reduce.java.opts</name>
      <value>-Xmx3278m</value>
   </property>
   <property> 
      <name>yarn.app.mapreduce.am.env</name>
	  <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property> 
    <property> 
       <name>mapreduce.map.env</name>
	    <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property> 
    <property> 
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property> 
</configuration>
```
### 3.4.7 The yarn-site.xml File

We use the text editor to add settings to the yarn-site.xml file.
```
$ nano hadoop-3.3.6/etc/hadoop/yarn-site.xml
```
We use the private IP address of the master node in this configure file:
```
<configuration>
   <property>
      <name>yarn.resourcemanager.hostname</name>
      <value>172.31.35.150</value>
   </property>
   <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
   </property>
</configuration>
```
### 3.4.8 The workers File

We should list all worker hostnames or IP addresses in the etc/hadoop/workers file, one per line. Therefore, the last step is to edit the file and add the private IP addresses of the three worker nodes to the file.
```
$ nano hadoop-3.3.6/etc/hadoop/workers
```
By default, the file only contains one hostname: localhost. We remove the default hostname and add these private IP addresses:

172.31.34.82
172.31.33.70
172.31.34.132
## 4 – Setting Up SSH Passwordless Login
After configuring the master node, we must copy the Hadoop installation on the master node to the three worker nodes. The SSH (Secure Shell) protocol can provide secure access for automated processes. We use this protocol to enable multiple computers to communicate. We generate and store the public key on the master node. We then copy the public key to the worker nodes to achieve passwordless SSH access across the cluster.

### 4.1 Generate a Pair of Public Key and Private Key on the Master Node
Run the following command on the master node to generate a key pair:
```
$ ssh-keygen -t rsa
```
The command prompts us for a file name and passphrase. We hit the Enter key to accept the default. We should know that adding a passphrase is essential for securing the private key (Smith, 2023). Since we want to create an environment for studying, we enter an empty value for the passphrase.
```
Enter file in which to save the key (/home/ec2-user/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```
The command generates these two files:

id_rsa: contains the private key.
id_rsa.pub: contains the public key.

### 4.2 Add the Public Key to the Master Node
Run the following command on the master node to add the public key to the node:
```
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
### 4.3 Copy the Public Key to Each Worker Node
On the master node terminal window, we run the following command to view the public key of the node:
```
$ cat  .ssh/id_rsa.pub
```
We want to copy the key in the .ssh/id_rsa.pub file and append it to the ~/.ssh/authorized_keys file on each worker node. Let us open a worker node terminal window and run the following command to open the authorized_keys in a text editor on the node:
```
$ nano ~/.ssh/authorized_keys
```
Then, we select and copy the public key on the master terminal window and then paste the key into the worker terminal window. Note that the public key should be in a single line. We may accidentally introduce a line break that prevents the key from matching its private key pair. Figure 24 exhibits the authorized_keys file on the worker terminal window. The first line is the public key of the .pem Amazon half. The second line is the public key we just copied from the master node terminal window. We should save the changes to the authorized_keys file.

We repeat the process and append the public key of the master node to all the worker nodes.

### 4.4 Access All the Worker Nodes from the Master Node
We need to make sure we can ssh to all worker nodes from the master node. Run the following command to access worker 1 from the master node:
```
$ ssh 172.31.34.82
```
We receive a command prompt that asks if we want to continue connecting. We enter yes to continue.

This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
We enter the exit command to exit the connection. We repeat the steps to connect the other two worker nodes:
```
$ ssh 172.31.33.70
$ ssh 172.31.34.132
```
## 5 – Setting Up the Worker Nodes
After setting up Hadoop on the master node, we copy the settings to the three worker nodes. First, we pack up the hadoop-3.3.6 folder on the master node. We then copy the package to the worker nodes. Next, we access each worker node, unpack the package, and install Java.

### 5.1 Creating a Hadoop Installation Package and Copy it to All Worker Nodes
Assuming we are in the root home directory, i.e., /home/ec2-user/. If not, we can run the cd command to change the current directory to the root directory. We then run the following command to pack up the entire hadoop-3.3.6 folder into a single file:
```
$ tar cvf Hadoop_Master.tar hadoop-3.3.6
```
Next, we run the following three commands to copy the file to the three worker nodes:
```
$ scp Hadoop_Master.tar ec2-user@172.31.34.82:/home/ec2-user/Hadoop_Master.tar 
$ scp Hadoop_Master.tar ec2-user@172.31.33.70:/home/ec2-user/Hadoop_Master.tar 
$ scp Hadoop_Master.tar ec2-user@172.31.34.132:/home/ec2-user/Hadoop_Master.tar
```
### 5.2 Unpacking the Hadoop Installation Package and Installing Java on Each Worker Node
We first use the ssh command to access worker 1 node:
```
$ ssh 172.31.34.82
```
We then run the following commands sequentially to unpack the Hadoop package and install Java on each node:
```
$ tar xvf Hadoop_Master.tar
$ sudo yum install java-1.8.0
$ sudo yum install java-1.8.0-devel
```
After completing the setting on one node, we run the exit command to exit the node. We then repeat these steps on the other two nodes. Here are all the commands to complete setting up the other two nodes:
```
$ ssh 172.31.33.70
$ tar xvf Hadoop_Master.tar
$ sudo yum install java-1.8.0
$ sudo yum install java-1.8.0-devel
$ exit
$ ssh 172.31.34.132
$ tar xvf Hadoop_Master.tar
$ sudo yum install java-1.8.0
$ sudo yum install java-1.8.0-devel
$ exit
```
## 6 – Running the Hadoop MapReduce Example on the Cluster
We start the Hadoop cluster and then run an example the Hadoop framework provides. The example is a simple MapReduce application that counts the number of occurrences of each word in a file.

### 6.1 Hadoop Startup
When using it for the first time, we must format Hadoop's distributed filesystem (HDFS) via the NameNode (Noll, 2011). Here is the command to format the new distributed file system:
```
$ hdfs namenode -format
```
This command creates a dfs folder in each node according to the settings in the hdfs-site.xml configure file. If, for some reason, we need to format the distributed file system again, we should delete the dfs folder in each node.

We then use this utility script to start all the HDFS processes:
```
$ start-dfs.sh
```
Next, we use the following utility script to start all the YARN processes:
```
$ start-yarn.sh
```
We use the jps command to verify that all processes start correctly.

We can also use the following command to verify that all three worker nodes started successfully:
```
$ hdfs fsck / -files -blocks
```
In addition, we can find the public IP address of the master node and access the web interface through this URL:

http://public IP address:9870
When switching to the Datanode tab, we find a list of worker nodes in the cluster.

### 6.2 Running a MapReduce Example on the Cluster
We copy text from the web page https://www.mssqltips.com/about/ to the nano text editor. Then, we save the text file on the master node. Here is the command we used:
```
$ nano Dataclustering
```
Next, we create a folder in the HDFS using this command:
```
$ hdfs dfs -mkdir /localfiles
```
Then, we copy the local text file to HDFS for processing:
```
$ hdfs dfs -put  Dataclustering/localfiles/
```
Finally, we run the Firefly_kmeans file:
```
$ time hadoop jar hadoop-3.3.6/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar wordcount /data/MSSQLTips /data/firefly_kmeans
```
We can use this command to view the output:
```
$ hadoop fs -cat /data/firefly_kmeans/part-r-00000 | more
```
### 6.3 Hadoop Shutdown
We use this utility script to stop all the HDFS processes:
```
$ stop-dfs.sh
```
Then, we use the following utility script to stop all the YARN processes:
```
$ stop-yarn.sh
```
