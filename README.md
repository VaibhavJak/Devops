# Devops-CI-CD-Pipeline
Built a Jenkins CI/CD pipeline to design and build, test and deploy a web application docker container to remote EC2 instance on AWS 

1)Created an EC2 instance on AWS

   - Created key pairs on AWS 
   - Downloaded private key locally (.pem file)
   - Public key stored by AWS

   Network Configuration:
      - Chose default VPC and subnet
      - Enabled public IP for our remote instance
      - Created security group and changed source type to "My IP"

   Kept the storage as default and launched instance

2)Connected to EC2 instance with ssh

   -Moved downloaded .pem file to .ssh subfolder in home folder:

     $ mv Downloads/docker-server.pem ~/.ssh/ 

   -Gave only read permission only to user:

     $ chmod 400 .ssh/docker-server.pem

   -"ssh"ed into remote instance using its public key, as ec2-user instead of default root user, using private key:

     $ ssh -i ~/.ssh/docker-server.pem ec2-user@15.236.202.138
    
3)Installed Docker on remote EC2 instance

     $ sudo yum update
     $ sudo yum install docker

   -checked the version installed:

     $ docker --version

   -Started docker daemon:
    
     $ sudo service docker start

   -checked if docker is actually running:

     $ ps aux | grep docker

   -Added ec2-user to docker group:
     
     $ sudo usermod -aG docker $USER

4)Ran Docker Container from private Repo

   -logged in with Docker ID to Docker Hub with the credentials:

     $ docker login

   -pulled and ran a docker web-app image from private dockerhub registry:

     $ docker pull docker pull vaibhavjak/va1bhav:latest
     $ docker run -d -p 3000:3080 vaibhavjak/va1bhav:latest

5)Configured EC2 Firewall to access App externally from browser

   -Edited inbound rules in security group of instance to allow inbound traffic from anywhere to port 3000 on instance

6)Jenkins pipeline job

   -Logged into Jenkins, and installed ssh-agent plugin from plugins

   Connected to EC2 in my multi-branch pipeline:

     -created new credential of type "SSH Username with private key" in my pipeline credentials scope by giving the instance username "ec2-user" and pasted contents of .pem file in passphrase.

     -Generated pipeline script by going to pipeline syntax in our multibranch pipeline and choosing SSH agent in the sample step and furthur choosing our "ec2-user" credential

     -Created a simple Jenkinsfile and added the generated script in the 'deploy' stage's script of the Jenkinsfile

     Configured Firewall on EC2:
      
        -Edited inbound rules to add Jenkins IP address as source for ssh 
        -Changed Custom TCP port to 3080 as our remote server is mapped to container on that port so that's where it needs to be accessed
     
   Built the feature branch in my pipeline and viewed the logs to find ssh into EC2 instance and docker run executed
   Verified by doing : "docker ps" in remote server to find the web-app container up and running.

   Hence, successfully deployed the application to EC2 server from Jenkins pipeline
