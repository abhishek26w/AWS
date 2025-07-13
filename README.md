Automate AWS EC2 Instance Launch with Shell Script 🚀"


📜 Script Overview
We’ll cover the following functions in the script:

Check if AWS CLI is installed

Install AWS CLI v2 (if not installed)

Create an EC2 instance

Wait until the instance is in the running state

✅ Step-by-Step Guide to Automate EC2 Launch Using Shell Script

📌 Prerequisites:
1️⃣An AWS account with IAM user and permissions to launch EC2.

2️⃣AWS CLI installed configured (aws configure)

3️⃣A Key Pair, Subnet ID, and Security Group ready.

4️⃣A Unix-based system (Ubuntu, WSL, macOS, etc.)

👉Steps:
Step 1: Check if AWS CLI is Installed
In the script, we use:

🛠️ Install AWS CLI ( If Missing )
The script checks if AWS CLI is installed. If not, it installs AWS CLI v2 automatically.

Manual Installation (Optional)
Here’s how you can do it:

Search on Internet:
Open your browser and type:
👉 aws cli v2
Hit enter.

Visit the Official AWS Page:
You’ll likely see the AWS documentation link at the top. Click the one that says something like:
✅ “Installing the AWS CLI version 2”

Choose Your Operating System:
Scroll down and find the section that matches your OS.
For example, I’m using Linux, so I clicked the Linux tab.

Follow the Linux Instructions:
The page will give you the latest download link and step-by-step commands to install the AWS CLI v2.

Here's the basic version if you’re using Ubuntu or WSL:


Copy

Copy
 curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
 sudo apt update && sudo apt install -y unzip
 unzip awscliv2.zip
 sudo ./aws/install
Check the Installation:
After it finishes, you can check if it worked by running:


Copy

Copy
 aws --version
You should see something like aws-cli/2.x.x confirming it's installed correctly.

If you already have AWS CLI installed but you're not sure which version, just run:


Copy

Copy
 aws --version
If it says version 2.x, you're good to go. If it's version 1.x, I highly recommend updating to version 2 — it has better support and newer features.

🛠️ Step 2: Set Up AWS Credentials
Run the following command in terminal:


Copy

Copy
aws configure
It will ask for:

AWS Access Key ID

AWS Secret Access Key

Default Region (e.g. us-east-1)

Default Output Format (e.g.json)

👉 Steps :








copy your Access key & password and paste your terminal.

also add region & output format ( your choice )

🛠️ Step 3: Prepare EC2 Configuration
Before running the script, make sure you have the following AWS values:

✅ AMI ID (e.g., Amazon Linux 2: ami-************)

✅ Instance Type (e.g., t2.micro)

✅ Key Pair Name

✅ Subnet ID

✅ Security Group ID

✅ Tag Name

👉 Steps :




🛠️ Step 3: Run the EC2 Automation Script
Save the script below as create_ec2.sh, make it executable, and run it:


Copy

Copy
chmod +x create_ec2.sh  
./create_ec2.sh or bash create_ec2.sh
📜 Full Script
🔧 Complete Bash Script to Automate EC2 Instance Creation

Copy

Copy
#!/bin/bash

set -euo pipefail

# Function to check if AWS CLI is installed

check_awscli() {

        if ! command -v aws &> /dev/null; then
                   echo "AWS CLI is not Installed. Please install it first." >&2

        return 1

        fi
         }
# Function to install AWS CLI v2

 install_awscli() {

                 echo "Installing AWS CLI v2 on Linux..."

# Download and Install AWS CLI v2

        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
        sudo apt-get update && sudo apt-get install -y unzip && \
        unzip awscliv2.zip && \
        sudo ./aws/install

# Verify Installation
  aws --version

# Cleanup
   rm -rf awscliv2.zip ./aws

  }

# Function to wait for an EC2 instance to reach the "running" state

waiting_for_instance() {

local instance_id="$1"
    echo "Waiting for instance $instance_id to be in Running state..."

while true; do

state=$(aws ec2 describe-instances --instance-ids "$instance_id" --query 'Reservations[0].Instances[0].State.Name' --output text)

        if [[ "$state" == "running" ]]; then
                 echo "Instance $instance_id is now running."
            break
        fi
        sleep 10
        done
     }

# Function to create an EC2 instance

create_ec2_instance() {

        local ami_id="$1"
        local instance_type="$2"
        local key_name="$3"
        local subnet_id="$4"
        local security_group_ids="$5"
        local instance_name="$6"

# Run AWS CLI command to create EC2 instance

instance_id=$(aws ec2 run-instances \
        --image-id "$ami_id" \
        --instance-type "$instance_type" \
        --key-name "$key_name" \
        --subnet-id "$subnet_id" \
        --security-group-ids "$security_group_ids" \
        --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=$instance_name}]" \
        --query "Instances[0].InstanceId" \
        --output text \
        )

    if [[ -z "$instance_id" ]]; then
           echo "Failed to create EC2 instance." >&2
                  exit 1
    fi

# Wait for the instance to be in running state                                                          
 waiting_for_instance "$instance_id"                                                                    
}                                                                                                       
# Main function                                                                                                                                                                                                 
main() {

        if ! check_awscli ; then
                install_awscli || exit 1
        fi

        echo "Creating EC2 instance..."

        AMI_ID=""
        INSTANCE_TYPE=""
        KEY_NAME=""
        SUBNET_ID=""
        SECURITY_GROUP_IDS=""
        INSTANCE_NAME=""
        create_ec2_instance "$AMI_ID" "$INSTANCE_TYPE" "$KEY_NAME" "$SUBNET_ID" "$SECURITY_GROUP_IDS" "$INSTANCE_NAME"

        echo "EC2 Instance Created Successfully!"

}
 # Be sure this is here and not missing (Run the Script)

 main "$@"
🔍 How the Script Works
Checks for AWS CLI → Installs if missing.

Launches an EC2 Instance → Uses run-instances with your config.

Waits for Instance to Start → Confirms it’s running before proceeding.



✅ Instance Running successfully
🎉 And That’s a Wrap!
Congratulations—you’ve just leveled up your cloud automation game! 🚀 No more manual clicking, no more waiting… just you, a cup of coffee ☕, and a script that does the work while you relax.






Subscribe to our newsletter
Read articles from DevOps Diaries directly inside your inbox. Subscribe to the newsletter, and don't miss out.

abhishek26w@gmail.com
Subscribe
Devops
Linux
DevOps Journey
Devops articles
shell script
AWS
ec2
Ubuntu
 
Article Series

Linux & SSH
1
Linux for Everyone
Introduction to Linux Linux is an operating system (OS) that works like the brain of a computer. It …

Linux for Everyone
2
"Master the Cloud: Set Up Your Ubuntu Linux EC2 Instance in 2 Minutes!"
Creating an Ubuntu Linux EC2 Instance on AWS This tutorial guides you through launching an Ubuntu Li…

"Master the Cloud: Set Up Your Ubuntu Linux EC2 Instance in 2 Minutes!"

Show all 3 posts
6
🚀 Shell Scripting in DevOps
Shell Scripting in DevOps Shell scripting automates repetitive tasks, an essential skill for DevOps …

🚀 Shell Scripting in DevOps
7
Automate AWS EC2 Instance Launch with Shell Script 🚀"
Have you ever wanted to spin up an EC2 instance without logging into the AWS Management Console? Whe…

Automate AWS EC2 Instance Launch with   Shell Script 🚀"
©2025 DevOps Diaries

Archive
·
Privacy policy
·
Terms
Powered by Hashnode - Build your developer hub.

Start your blog
Creat
