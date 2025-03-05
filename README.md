aws_wq
# World Quant Brain Alpha Simulation on AWS EC2
## Description
### This project demonstrates how to deploy a Python-based simulation environment for the World Quant Brain Alpha framework on AWS EC2. The setup includes configuring a Jupyter Notebook server for remote analysis and ensuring secure, scalable cloud resource management.

## Prerequisites
An AWS account with IAM permissions to create users, EC2 instances, and security groups.
Basic knowledge of Linux, SSH, and AWS CLI.
Setup Steps
1. Configure AWS IAM User & Key Pair
bash
# Create an IAM user with EC2FullAccess policy (least privilege alternative: EC2ReadOnlyAccess)
aws iam create-user --name quant_user
aws iam attach-user-policy --user-name quant_user --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess

# Generate a key pair (replace `quant-keypair.pem` with your desired name)
aws ec2 create-key-pair --key-name quant-keypair --query 'KeyMaterial' --output text > quant-keypair.pem
chmod 400 quant-keypair.pem
⚠️ Security Note: Avoid using the root account for EC2 access. Always use IAM users with restricted permissions.

2. Launch EC2 Instance
Choose an Instance:
Free-tier eligible: t2.micro (Linux) or t3.nano
AMI: Amazon Linux 2 LTS (ami-0c55b159cbfafe1f0)
Configure Security Group:
Add inbound rules for:
SSH (22) from your IP (or 0.0.0.0/0 for testing)
Jupyter Notebook (8888)
3. Connect to EC2 Instance
bash
ssh -i quant-keypair.pem ec2-user@<your_instance_public_ip>
⚠️ Note: Replace <your_instance_public_ip> with your actual IP (e.g., ec2-3-133-131-201.us-east-2.compute.amazonaws.com).
🔗 Check your instance public IP in the AWS Console.

4. Set Up Python Environment
Update System
bash
sudo yum update -y
Install Miniconda
bash
# Download Miniconda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

# Install
bash Miniconda3-latest-Linux-x86_64.sh
# Follow prompts, install to default location (/home/ec2-user/anaconda3)

# Disable auto-activation (optional)
conda config --set auto_activate_base false
Cleanup
bash
# Free up space
sudo yum clean all
conda clean --all
df -h  # Verify disk usage
5. Configure Jupyter Notebook Server
Install Jupyter
bash
conda install -y notebook
Generate Password
python
from jupyter_server.auth import passwd
password = passwd("your_strong_password")
print(f"Password hash: {password}")
Update Configuration File
bash
jupyter notebook --generate-config
Edit ~/.jupyter/jupyter_notebook_config.py:

python
c.NotebookApp.ip = '*'          # Listen on all interfaces
c.NotebookApp.port = 8888      # Custom port
c.NotebookApp.open_browser = False
c.NotebookApp.password = 'sha1:67c9e60bb8b6:9ffede0825894254b2e042ea597d771089e11aed'  # Replace with your hash
c.NotebookApp.allow_remote_access = True
Security Best Practices
bash
chmod 600 ~/.jupyter/jupyter_notebook_config.py  # Protect config file
6. Start & Access Jupyter Notebook
bash
# Start Jupyter in background (to keep it running after logout)
nohup jupyter notebook &
Access via Web Browser
Navigate to: http://<your_instance_public_ip>:8888
Enter the password generated earlier.
Troubleshooting
Connection Issues: Verify security group inbound rules for ports 22 and 8888.
Password Errors: Re-run the password generation step and update the config file.
Instance Reboots: The public IP may change; use a DNS name (e.g., ec2-<region>.compute.amazonaws.com) or Elastic IP for persistence.
Alternatives & Enhancements
Use Docker: containerize the environment with docker-compose.yml.
Auto-Scaling: Configure an Auto Scaling Group for high availability.
Cost Optimization: Use Spot Instances or Reserved Instances for long-term simulations.
Let me know if you'd like further refinements! 🚀
