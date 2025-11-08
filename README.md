🚀 DevOps Cloud Automation Project (Terraform, Ansible, Docker, GitHub Actions)
🧩 PART 1 — Infrastructure Setup (Terraform + AWS)

🎯 Purpose:
To automate provisioning of AWS infrastructure using Infrastructure as Code (IaC) with Terraform.

🛠️ AWS Services Used:

EC2 Instance – compute node for deployment

VPC + Subnet – secure network segmentation

Security Group – allow inbound traffic on ports 80 (HTTP) and 22 (SSH)

Key Pair – for secure SSH access

📂 Folder Structure:

project/
 └── terraform/
      ├── variables.tf
      ├── main.tf
      ├── outputs.tf


⚙️ Steps to Provision Infrastructure:

# 1. Install Terraform
sudo yum install -y terraform
terraform -v

# 2. Configure AWS credentials
aws configure

# 3. Initialize Terraform
cd terraform
terraform init

# 4. Plan and apply
terraform plan
terraform apply -auto-approve


✅ Output:
Terraform provisions the EC2 instance and prints its public IP.
You can then SSH into it using:

ssh -i aws-key.pem ec2-user@<EC2_PUBLIC_IP>

⚙️ PART 2 — Configuration Management (Ansible or Equivalent)

🎯 Purpose:
Automate EC2 server configuration — install Docker and enable it to auto-start on boot.

📁 Ansible Inventory Example:

[webserver]
<EC2_PUBLIC_IP> ansible_user=ec2-user ansible_ssh_private_key_file=../aws-key.pem


▶️ Run Playbook:

cd ansible
ansible-playbook -i inventory playbook.yaml


🧾 Expected Outcome:

Docker installed successfully

Docker service enabled on startup

EC2 user added to Docker group

💻 Equivalent Manual Commands (if not using Ansible):

sudo yum update -y
sudo yum install docker -y
sudo service docker start
sudo service docker status
sudo usermod -aG docker ec2-user
sudo reboot


🧰 Verification:

docker version
docker ps


🐳 Docker Test Run:

docker run -d -p 80:80 nginx
docker ps
docker logs <container_id>


✅ Docker is now running and ready for container deployment.

🐳 PART 3 — Docker Container Deployment

🎯 Purpose:
Containerize and deploy a simple web application on the EC2 instance.

📂 Folder Structure:

app/
 ├── Dockerfile
 ├── index.html


🧱 Steps:

# Connect to EC2
ssh -i aws-key.pem ec2-user@<EC2_PUBLIC_IP>

# Setup environment
mkdir myapp && cd myapp
echo "Welcome to My Docker Web App - Hosted on AWS EC2" > index.html

# Create Dockerfile
vi Dockerfile
# (insert the following)
FROM nginx:latest
COPY . /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
:wq

# Build and run
docker build -t myapp .
docker run -d -p 80:80 myapp


🔍 Verify:

docker ps


Visit your application at:
👉 http://<EC2_PUBLIC_IP>
You’ll see:

“Welcome to My Docker Web App – Hosted on an EC2 instance using Docker!”

🔁 PART 4 — CI/CD Pipeline (GitHub Actions)

🎯 Purpose:
Automate continuous integration and deployment using GitHub Actions.

⚙️ Workflow:
Whenever new code is pushed to GitHub:

GitHub Actions pipeline is triggered

Terraform provisions infrastructure

Ansible installs and configures Docker

Application is automatically deployed

📂 Pipeline Structure:

.github/workflows/
 └── deploy.yml


🧩 Example deploy.yml:

name: CI/CD Pipeline
on:
  push:
    branches: [ "main" ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v2

      - name: Terraform Init & Apply
        run: |
          cd terraform
          terraform init
          terraform apply -auto-approve

      - name: Deploy Docker App
        run: |
          ssh -o StrictHostKeyChecking=no ec2-user@${{ secrets.AWS_EC2_IP }} \
          "sudo docker build -t myapp /home/ec2-user/myapp && sudo docker run -d -p 80:80 myapp"


🧭 GitHub Setup Commands:

git init
git add .
git commit -m "DevOps Project Initial Commit"
git branch -M main
git remote add origin <your-repo-URL>
git push -u origin main


✅ Expected Result:
Every push automatically triggers the build → deploy pipeline, ensuring full Continuous Integration and Continuous Deployment.

🧠 End-to-End Workflow Summary
Stage	Tool	Purpose
1. Infrastructure	Terraform	Provision AWS EC2, VPC, Security Groups
2. Configuration	Ansible / Shell	Install and configure Docker automatically
3. Deployment	Docker	Build and run web app container
4. Automation	GitHub Actions	CI/CD pipeline triggers automatic deployment
💡 Final Outcome

A fully automated DevOps pipeline that provisions cloud infrastructure, configures the environment, deploys a containerized web app, and continuously integrates and delivers updates through GitHub.