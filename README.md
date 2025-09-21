# Canary Based Slow Rollout in AWS with GitLab and Runner CI

## 1) GitLab and Runner
### Step 1: Create an EC2 instance in the default VPC in AWS in any region (for e.g. North Virginia) of at least the following Spec:- 
- OS: Ubuntu 24.04 LTS
- Insance Type: t2.medium (2vCPU, 4GB RAM)
- Key Pair: Create a new key pair and download
- Security Group:- Create a new security group and allw SSH, HTTP & HTTPS from anywhere (recommended just for demo purpose only)
- Storage:- 10GiB

### Step 2: Associate an Elastic IP to the EC2 instance
- EC2 > Instances > Network & Security > Elastic IPs > Allocate Elastic IP Address >  Amazon's pool of IPv4 addresses > Network border group: us-east1 > Allocate
- Edit the Name for Elastic IP address and give it a name (for e.g. GitLab)
- Actions > Associate Elastic IP Addres > Choose Instance ID > Associate

### Step 3: Update DNS 
- Go to the DNS provider of your domain and update the DNS record for the GitLab domain/subdomain with the Elasti IP you have associated just before.
NOTE: If you are using CloudFlare make sure you turn of the proxy (set DNS only)

### Step 4: Access the VM (using SSH)
- Go to the directory where you have downloaded the .pem key. Change the permission of the key to '400'. For e.g.
```
cd Downloads
chmod 400 gitlab-key.pem
ssh -i "gitlab-key.pem" ubuntu@ec2-98-85-13-167.compute-1.amazonaws.com
```
Type 'yes' in the prompt and hit enter.

- Run the following basic commands in the EC2 instance terminal
```
sudo apt update -y && sudo apt upgrade -y
```
### Step 5: Install GitLab on the EC2 instance along with LetsEncrypt SSL
- Installation guide : https://docs.gitlab.com/install/package/ubuntu/?tab=Enterprise+Edition
- Installation steps:-
  ```
  sudo apt install -y curl
  curl "https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh" | sudo bash
  sudo EXTERNAL_URL="https://gitlab.swiftpearls.com" apt install gitlab-ee
  ```
- Get the Initial root password to login to the GitLab
  ```
  sudo cat /etc/gitlab/initial_root_password
  ```
- Create an Admin User and remove the default root user (Optinal but Recommended)
  - Login to the GitLab withe the initial password and the root user > Admin > Users > Add New
  - After the new user is created edit the user and set password. Users > New User > Edit > Password > Enter the password and save.
  - Logout of the root user and login with the new admin user. Delete the old user. Steps:- Gitlab > Admin > Uses > Root user > click n the three vertical dots > Delete User.
    
### Step 6: Configure a GitLab Runner and register it with the GitLab
- GitLab Dashboard > Admin > CI/CD > Runners > Create Instance Runner
  - Tags : Tick on 'Run untagged jobs'
  - Leave all the others fields empty and all the other checkbox unchecked and click on the 'Create Runner' button.
  - SSH to the same server where GitLab is installed (for demo purpose only. For production configure separate server). Then run the commands as specified in Step1. For e.g.
    ```
    ssh -i "gitlab-key.pem" ubuntu@ec2-98-85-13-167.compute-1.amazonaws.com
    
    sudo gitlab-runner register  --url https://gitlab.swiftpearls.com  --token glrt-S5shDi7RTLnf09ue85dTsm86MQp0OjEKdTozCw.01.120xjpoan
    ```
    - If you get ``` sudo: gitlab-runner: command not found ```, click on the 'How do I install GitLab Runner?' link just above the Step1 and then copy and paste the commands. Then run the above command again.
    - ```
      Enter the GitLab instance URL (for example, https://gitlab.com/):
      [https://gitlab.swiftpearls.com]: <Just hit enter>
      Enter a name for the runner. This is stored only in the local config.toml file:
      [ip-172-31-29-77]: <Just hit enter>
      Enter an executor: custom, ssh, parallels, docker, docker+machine, kubernetes, instance, shell, virtualbox, docker-windows, docker-autoscaler: docker
      Enter the default Docker image (for example, ruby:3.3): roma599/slowrollout-ci:latest
      ```
      [You will see the 'Runner registered successfully' message]
  - To verify, Click on View Runner. Alternatively, Gitlab Dashboard > Admin > CI/CD > Runners 
    You may see the Status as 'online/idle'. That's good.

### Step 7:  Add SSH Key, create a repo and commit the demo app files 
- From your local PC's terminal, Generate a SSH Key Pair using the following commands:-
  ```
  $ cd ~/.ssh/
  ssh-keygen -t rsa -b 2048 -C "swiftpearls"
  Generating public/private rsa key pair.
  Enter file in which to save the key (/home/sanjaya-regmi/.ssh/id_rsa): swiftpearls
  <just keep hitting enter>
  $ cat swiftpearls.pub
  <Copy the key somewhere in your PC for e.g. Notepad>
  ```
- Add the SSH pubic key to GitLab.
  - GitLab > Your Profile icon > Preferences > SSH Keys > Add New key > Paste the key from Notepad to box. Leave all the other fields auto filled. Remove the Expiration date. > Add Key

- Create a new repo. First create a new group.
  - GitLab Dashboard > Admin > Groups > New Group > Create Group. Give a name for e.g. demos. Keep it private > select 'My company or tem' > Create Group
  - For new repo in the group: New Project > Create blank Project > Give a project name such as 'Canary Slow Rollout Demo'. Keep easy Project Slug such as 'slow-rollout-demo' > Check 'Initialize repository with a README' > Create Project

- Clone the repo.
  - Go to the repo > Code > copy the link/command in the 'Clone with SSH' box.
  - Go to your PC's terminal and clone the repo.
    ```
    git clone git@gitlab.swiftpearls.com:demos/slow-rollout-demo.git
    ```
  - Copy your demo application file to you newly cloned repo (in local). Don't commit and push.

  ### Step 8: Create EKS cluster in AWS
  - Install AWS CLI depending upon your Operating System by following the AWS documentation:- https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
  - Create a AWS User and use it's credentials to connect to the AWS by using AWS CLI.
    - IAM > Create User > Give a user name for e.g. roma. Check 'Provide user ......'. Make sure 'I want to create an IAM user ...' is selected > Next > Select 'Autogenerated password' and 'Users must create a new pasword at next sign-In - Recommended' > Next > Attach policies directly > Check 'AdministratorAccess' > Next > Create User > Download .csv file > Return to users list.
    - Click on the newly created user (i.e. roma) > Security credentials > Access Keys > Create access key > Use case: CLI > Check on 'I understand ...' > Next > Create access key. Note down the Access key and the secret access key. Done.
    - Configure your Local machine to authenticate to AWS using AWS CLI
      ```
      $ aws configure
      Enter the access key, aws secret key, default region as 'us-east-1' and default output as blank.
      
      # Verify with the following command
      $ aws ec2 describe-instances --region us-east-1 --query 'Reservations[].Instances[].{ID:InstanceId,State:State.Name}' --output table 
      ```
  - Create an EKS cluster with 2 worker nodes each of 4GB or RAM and 2vCPUs.
    - Install the tool 'eksctl'. The installation method can be different depending upon your Operating System. For official guide, refer to the link:- https://eksctl.io/installation/
    - Create a cluster as per your requirement using the eksctl command. For e.g.
       ```
       # Create Cluster
       eksctl create cluster --name pm-policies-cluster --region us-east-1 --node-type t3.medium --nodes 2
       NOTE: It may take 10-15 minutes to complete the cluster creation
       
       # Check the status
       eksctl get cluster 
       eksctl get cluster --name pm-policies-cluster --region us-east-1
       eksctl get cluster --name pm-policies-cluster --region us-east-1 -o yaml
      ```
  - Check the status of the cluster
    ``` aws eks describe-cluster --name slow-rollout-cluster --region us-east-1 --query 'cluster.status' --output text  ```

  - Stop the cluster (to save billing)
  - Delete the cluster
    

  
    

