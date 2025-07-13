**Automate AWS EC2 Instance Launch with Shell Script 🚀"**


**📜 Script Overview**

We’ll cover the following functions in the script:

1️⃣ Check if AWS CLI is installed

2️⃣ Install AWS CLI v2 (if not installed)

3️⃣ Create an EC2 instance

4️⃣ Wait until the instance is in the running state


**✅ Step-by-Step Guide to Automate EC2 Launch Using Shell Script**


📌 Prerequisites:

1️⃣An AWS account with IAM user and permissions to launch EC2.

2️⃣AWS CLI installed configured (aws configure)

3️⃣A Key Pair, Subnet ID, and Security Group ready.

4️⃣A Unix-based system (Ubuntu, WSL, macOS, etc.)

👉Steps:

**Step 1: Check if AWS CLI is Installed**

In the script, we use:

🛠️ Install AWS CLI ( If Missing )
The script checks if AWS CLI is installed. If not, it installs AWS CLI v2 automatically.

Manual Installation (Optional)
Here’s how you can do it:

1. Search on Internet:
Open your browser and type:
👉 aws cli v2
Hit enter.

2. Visit the Official AWS Page:
You’ll likely see the AWS documentation link at the top. Click the one that says something like:
✅ “Installing the AWS CLI version 2”

3. Choose Your Operating System:
Scroll down and find the section that matches your OS.
For example, I’m using Linux, so I clicked the Linux tab.

4. Follow the Linux Instructions:
The page will give you the latest download link and step-by-step commands to install the AWS CLI v2.

Here's the basic version if you’re using Ubuntu or WSL:


Copy COMMAND:

 curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
 sudo apt update && sudo apt install -y unzip
 unzip awscliv2.zip
 sudo ./aws/install

5. Check the Installation:
After it finishes, you can check if it worked by running:

 aws --version
 
You should see something like aws-cli/2.x.x confirming it's installed correctly.

6. If you already have AWS CLI installed but you're not sure which version, just run:

 aws --version
 
If it says version 2.x, you're good to go. If it's version 1.x, I highly recommend updating to version 2 — it has better support and newer features.


**🛠️ Step 2: Set Up AWS Credentials**

Run the following command in terminal:

aws configure

It will ask for:

. AWS Access Key ID

. AWS Secret Access Key

. Default Region (e.g. us-east-1)

. Default Output Format (e.g.json)

👉 Steps :


![a4d78bc3-a4e9-4664-8a59-e9cbfd1397c7](https://github.com/user-attachments/assets/c54357ac-da7f-4b97-8ba8-b3cd9749575a)


![d467a360-03c2-4f3e-ae56-37e376e78e45](https://github.com/user-attachments/assets/a8444433-33ea-4aae-b328-694cf1364c87)


![b710d1e9-5dd4-4d3c-b4ba-7cf9e1c99e9a](https://github.com/user-attachments/assets/59a83676-b842-4ffd-bf26-4cbc9b97e166)


![d0ca2584-bd90-415e-bea7-8f287d8f4154](https://github.com/user-attachments/assets/ce0371dd-a9d4-43de-a645-4bcd5d7fa949)



copy your Access key & password and paste your terminal.

also add region & output format ( your choice )



**🛠️ Step 3: Prepare EC2 Configuration**

Before running the script, make sure you have the following AWS values:

✅ AMI ID (e.g., Amazon Linux 2: ami-************)

✅ Instance Type (e.g., t2.micro)

✅ Key Pair Name

✅ Subnet ID

✅ Security Group ID

✅ Tag Name

👉 Steps :

![b73b0a94-2c6c-4cf8-8b02-1562002bab7d](https://github.com/user-attachments/assets/a7aafa2c-af96-4d82-91ff-d0f4dac2fba4)


![dfa87577-0d72-4f1c-abde-3f97941d5c8c](https://github.com/user-attachments/assets/1468292f-08f1-42f6-ab5d-d45560db950c)


**🛠️ Step 4: Run the EC2 Automation Script**
Save the script below as create_ec2.sh, make it executable, and run it:


chmod +x create_ec2.sh  
./create_ec2.sh or bash create_ec2.sh


📜 Full Script
🔧 Complete Bash Script to Automate EC2 Instance Creation

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


 
**🔍 How the Script Works**

1. Checks for AWS CLI → Installs if missing.

2. Launches an EC2 Instance → Uses run-instances with your config.

3. Waits for Instance to Start → Confirms it’s running before proceeding.

![d260174b-f7bf-40c8-a818-c9ff6d716dde (1)](https://github.com/user-attachments/assets/dca1a431-7825-4561-a5bd-1f252f335a4a)


✅ Instance Running successfully
🎉 And That’s a Wrap!

Congratulations—you’ve just leveled up your cloud automation game! 🚀 No more manual clicking, no more waiting… just you, a cup of coffee ☕, and a script that does the work while you relax.





