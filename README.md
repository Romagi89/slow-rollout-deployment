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
