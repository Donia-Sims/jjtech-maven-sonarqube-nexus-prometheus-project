METHOD 1

# spin up 3 linux ec2 instances, instance type:t2.medium, ami: aws linux2 AMI
# Install jenkins in one of the servers considered as master

# Install Java on the two slave nodes

```sudo amazon-linux-extras install java-openjdk11 -y```

# Install git on slave nodes

```sudo yum install git -y```

# Login to jenkins
- piblic_ip:8080
- create default user
- navigate to jenkins dashboard >> manage jenkins >> nodes >> New Node
- Provide node name >> check Permanent Agent >> create
- use /opt/jenkins-builds as  Remote root directory
- provide a label to the agent e.g Linux server
- under "Custom WorkDir path" use your remote root directory set above
- Check "Use WebSocket"
- Save the settings

# Run the following commands in the jenkins slave nodes
# Customize the jenkins run command to connect jenkins master to slave. Ensure jenkins public IP matches your current public IP
# Add sudo infront of the command and add "&" to run in background mode

```sudo curl -o agent.jar http://100.26.189.4:8080/jnlpJars/agent.jar```

```java -jar agent.jar -jnlpUrl http://3.145.136.229:8080/computer/linux%2Dslave1/jenkins-agent.jnlp -secret 075ae5cebbb7e6e83f70ca3195cf57183aabac754e81c55032a19ba85cbc0dd3 -workDir "/opt/jenkins-builds" &```

##########################################################################################################
#######################################################################################################



METHOD 2 (Using SSH)

Pre-requisites:

- Jenkins Master is already setup and running
- Create a new EC2 instance for Slave

Steps involved:
1. Setup new EC2 instance for slave
2. Create jenkins user and Install Java, Maven in Slave node
3. Create SSH keys in jenkins master and upload public keys from master to slave node.
4. verify ssh connection from master to slave
5. Register slave node in Jenkins master
6. Run build jobs in Jenkins slave

############################################################################################

# Slave node configuration

(You need to create at least t2.micro Ubuntu 22.0.4 instance for this slave)
only port 22 needs to be open

# Change Host Name to Slave
```sudo hostnamectl set-hostname jenkins-agent2```

# Install Java
```sudo apt update -y ```
```sudo amazon-linux-extras install java-openjdk11 -y```

# Install Maven
1. Setup and connect to an Amazon EC2 linux2 instance with an SSH client. Note-Do not select the amazon 2023 instance. Choose the linux2 ami instead. 

2. Install Apache Maven on your EC2 instance. First, enter the following to add a repository with a Maven package.

    ```
    sudo wget https://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo
    ```

- Enter the following to set the version number for the packages.

    ```
    sudo sed -i s/\$releasever/6/g /etc/yum.repos.d/epel-apache-maven.repo
    ```
- Then you can use yum to install Maven.

    ```
    sudo yum install -y apache-maven
    ```
3. The Gremlin libraries require Java 8. Enter the following to install Java 8 on your EC2 instance.

    ```
    sudo yum install java-1.8.0-devel -y
    ```
4. Install Java11 for SonarQube
   ```
   sudo amazon-linux-extras install java-openjdk11
   ```
   or
   
   ```
   sudo dnf install java-11-amazon-corretto -y
   ```
6. Enter the following to set Java 8 as the default runtime on your EC2 instance.

    ```
    sudo /usr/sbin/alternatives --config java
    ```
- When prompted, enter the number `4` for Java 11.

5. Enter the following to set Java 8 as the default compiler on your EC2 instance.

    ```
    sudo /usr/sbin/alternatives --config javac
    ```
- When prompted, enter the number `2` for Javac maven compiler.
- Make sure to review this config if `mvn compile` breaks

6. Verify your maven version
    ```
    mvn -v

# Create User as Jenkins
```sudo useradd -m jenkins```

```sudo -u jenkins mkdir /home/jenkins/.ssh```

################################################################################################

# Now Login to Jenkins Master through ssh
Create SSH keys by executing below command:
```ssh-keygen -t rsa -m PEM```

# Copy SSH Keys from Master to Slave 

Execute the below command in Jenkins master EC2.
```sudo cat ~/.ssh/id_rsa.pub```

###############################################################################################

# Now Login to Slave node and execute the below command:
```sudo -u jenkins vi /home/jenkins/.ssh/authorized_keys```

This will be empty file, now copy the public keys from master into above file. 
Once you pasted the public keys in the above file in Slave, come out of the file by entering :wq! or SHIFT+ZZ

#################################################################################################

# Now go into master jenkins server and run the command: <span style="color:red;">REPLACE SLAVE IP ADDRESS in comand below</span>
```ssh jenkins@slave_node_ip  ```

This is to make sure master is able to connect to slave node. once you are successfully logged into slave, type exit to come out of slave.

################################################################################################

# In Jenkins app:

Now to go Jenkins Master, Dashboard, manage jenkins, manage nodes.

- Click on new node. give name and check permanent agent.
- give name and no of executors as 1. enter /home/jenkins as remote directory.
- select launch method as Launch slaves nodes via SSH.
- enter Slave node ip address as Host.

- click on credentials. Enter user name as jenkins.
- Kind as SSH username with private key. enter private key of master node directly by executing below command on jenkins master server:
  sudo cat ~/.ssh/id_rsa
- Select Manaually trusted key verification

#########################################################################################################

Now you can kick start building the jobs, you will see Jenkins master runs jobs in slave nodes.

###############################################################################################################

ENABLE GITHUB WEBHOOK

- Sing in to your gu=ithub account
- Navigate to the repository that contains your source code
- Go to settings >> Webhooks >> add webhook
- In Payload URL section, enter http://JENKINS_IP:8080/github-webhook/
- content type should be "application/json"
- ensure "just push event" is checked
- Add webhook
- Verify connection is established by agreen tick







