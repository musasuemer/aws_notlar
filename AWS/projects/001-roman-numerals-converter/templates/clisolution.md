WARNING!!!

- Ok! First of all, we need to launch an EC2 instance which has Amazon Linux 2 AMI to execute commands in on hand. Because, commands can change based on operating systems. We'll attach security group which allows ssh from anywhere.

- We should update yum package and install AWS CLI v2. (for more information to install AWS CLI v2 please look at https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)

```bash
sudo yum update -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

- Write your credentials using this command

```bash
aws configure
```

aws sts get-caller-identity --query Account --output text

1. Create Security Group

```bash
aws ec2 create-security-group \
    --group-name ms_roman_numbers_converter_sec_grp \
    --description "This Sec Group is to allow ssh and http from anywhere"
```

- We can check the security Group with these commands

```bash
aws ec2 describe-security-groups --group-names ms_roman_numbers_converter_sec_grp
```

2. Create Rules of security Group

```bash
aws ec2 authorize-security-group-ingress \
    --group-name ms_roman_numbers_converter_sec_grp \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-name ms_roman_numbers_converter_sec_grp \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

3. After creating security Groups, We'll create our EC2 which has latest AMI id. to do this, we need to find out latest AMI with AWS system manager (ssm) command

- This command is to get description of latest AMI ID (Linux 2) that we use.

```bash

aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --region us-east-1
```

- This command is to run querry to get latest AMI ID (Linux 2)

```bash

aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text
```

- we'll assign this latest AMI id to the LATEST_AMI environmental variable

```bash
LATEST_AMI=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text)
```

- Now we can run the instance with CLI command. (Do not forget to create userdata.sh under "/home/ec2-user/" folder before run this command)

```bash
touch userdata.sh
vim userdata.sh
#! /bin/bash
yum update -y
yum install python3
pip3 install flask
yum install git -y
cd /home/ec2-user
wget -P templates https://raw.githubusercontent.com/okti3535/aws-projects-de07/main/001-roman-numerals-converter/templates/index.html
wget -P templates https://raw.githubusercontent.com/okti3535/aws-projects-de07/main/001-roman-numerals-converter/templates/result.html
wget https://raw.githubusercontent.com/okti3535/aws-projects-de07/main/001-roman-numerals-converter/app.py
python3 app.py
```

```bash
aws ec2 run-instances --image-id $LATEST_AMI --count 1 --instance-type t2.micro --key-name first_key --security-groups ms_roman_numbers_converter_sec_grp --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=ms_roman_numbers}]' --user-data file:///home/ec2-user/userdata.sh
or

aws ec2 run-instances \
    --image-id $LATEST_AMI \
    --count 1 \
    --instance-type t2.micro \
    --key-name okt-aws \
    --security-groups okde07_roman_numbers_converter_sec_grp \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=okt_roman_numbers}]' \
    --user-data file:///home/ec2-user/userdata.sh
```

- To see the each instances Ip we'll use describe instance CLI command

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=
okt_roman_numbers"
```

- You can run the query to find Public IP and instance_id of instances:

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=okt_roman_numbers" --query 'Reservations[].Instances[].PublicIpAddress[]'

aws ec2 describe-instances --filters "Name=tag:Name,Values=okt_roman_numbers" --query 'Reservations[].Instances[].InstanceId[]'
```

- To delete instances

```bash
aws ec2 terminate-instances --instance-ids <We have already learned this id with query on above>
```

- To delete security groups

```bash
aws ec2 delete-security-group --group-name okde09_roman_numbers_converter_sec_grp
```

# aws ec2 create-tags --resources i-5203422c --tags Key=Name,Value=MyInstance

# https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html

# https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/run-instances.html

# ResourceType=string,Tags=[{Key=string,Value=string},{Key=string,Value=string}]

# https://aws.amazon.com/blogs/compute/query-for-the-latest-amazon-linux-ami-ids-using-aws-systems-manager-parameter-store/

aws ec2 run-instances --image-id ami-xxxxxxxx --count 1 --instance-type t2.micro --key-name MyKeyPair --security-group-ids sg-903004f8 --subnet-id subnet-6e7f829e
