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
  - Logout of the root user and login with the new admin user. Delete the old user. Gitlab > Admin > Uses > Root user > Edit > 
    

