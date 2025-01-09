## Step 1: create a keypair

##### The first thing you have to do is to create a new keypair. So in the AWS management console go to “EC2” and select “Key pairs” in the listed overview of your resources:

![image](https://github.com/user-attachments/assets/1c68ddc9-7826-40c7-83bb-d93e074d0934)

##### Then select “Create key pair” on the top-right corner.

![image](https://github.com/user-attachments/assets/a58dc44d-da0e-482c-9d1d-6c1cf1e2e5dc)

##### Give a name to the keypair and select as key file format “.pem”, then select “Create key pair” and save it in a new folder for this project, if you haven’t created it yet.

## Step 2: create the projects folders

##### In your project folder you can follow the structure of the code of my repository, you can also directly fork the repository if you prefer

##### The structure is the following:

![image](https://github.com/user-attachments/assets/93a59b50-939b-4589-af54-5d9c766fb2a0)

##### Now I will dive in and explain the content each folder.

## Step 3: setup Jenkins server

##### Here I will explain what the code inside “1-terraform-jenkins-server” does.

##### However the first thing that we need to do is to create an S3 bucket that will be used as the Terraform state file storage. Make sure to be consistent with the AWS region you choose, for example for all the project I will always use “eu-west-3”, so I’m creating an S3 bucket on this region. If you plan to use another AWS region, ensure to replace “eu-west-3” anywhere in the code with your region.

##### So we create the bucket and ensure that we choose a unique name for the bucket, I will call it “jenkins-app-kub-2024-v2” but choose a different name.

![image](https://github.com/user-attachments/assets/f0070f65-3a9a-4d11-aa90-2d3b51dd24f2)

##### Now let’s go back to the folder ““1-terraform-jenkins-server”.

##### In the file “provider.tf” we ensure that Terraform uses the specified version of the AWS provider and operates within the “eu-west-3” region when interacting with AWS resources. 

##### In “variables.tf” we define all the variables that we need throughout the proget

##### And then in “terraform.tfvars” we provide the values that we want to assign to the variables

##### In the file “backend.tf” we configure the backend to use the S3 bucket created before, where Terraform can store its state file centrally. The state file keeps track of the resources that Terraform manages and their current state. This is also important for collaboration among team members and ensuring consistent infrastructure management

##### In “server.tf” we define an Amazon Linux Image (AMI) and the EC2 instance that will use it. The EC2 instance is our Jenkins server

##### In “jenkins-script.sh” we define a list of commands to install and start all the required services and packages

##### In “outputs.tf” we tell Terraform that we want to print to the console the address of the EC2 instance once it is launched

##### In “vpc.tf” we define the VPC and a private subnet

##### In “route.tf” we define an Internet Gateway and the default route table for our VPC:

##### The Internet Gateway allows communication between the EC2 instance (the Jenkins server) in your VPC and the Internet, without this your EC2 instance in your subnet would not have access to the Internet

##### A default route table is automatically created for our VPC, here we are associating the Internet Gateway with the default route (route block) which states: “Any traffic destined to the Internet (0.0.0.0/0) will be routed through the Internet Gateway”.

##### In “security.tf” we define a Security Group for our Jenkins server, allowing SSH access 

##### We have finished commenting this part of the project. Now, if you haven’t already done it, deploy this infrastructure by running these commands in the terminal, make sure you are in the right folder (jenkins-server):

terraform init

terraform plan

terraform apply

##### After running all the commands, make a note of the public IP address of your Jenkins server, which will be printed in the console.

##### Now, paste this IP address into your web browser’s address bar, followed by ‘:8080’. The Jenkins server welcome page should show up.

![image](https://github.com/user-attachments/assets/aafbff80-f29b-4825-9db3-80176a4b2650)

##### To access to our Jenkins server, we need a password. So we first connect to our EC2 instance through SSH, make sure that your terminal is opened in the same folder as the key you downloaded before (.pem file) when creating the key pair. So, run the following command to connect to the EC2 instance:

ssh -i "jenkins-server.pem" ec2-user@1.234.567.89

##### In the above command, ensure that you replace ‘jenkins-server.pem’ with the name of your key pair and “1.234.567.89” with the public IP you noted earlier.

##### After running it, type “yes” and press Enter.

##### Once you are connected to the EC2 instance, run:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

![image](https://github.com/user-attachments/assets/ac615af2-cbc1-423b-a6e2-95e9c93a58a7)

#####You will get a password. Copy it and then paste it in the field “Administrator password” in the Jenkins welcome screen and then click on Continue:

![image](https://github.com/user-attachments/assets/1c6dcd23-f588-4730-badf-cb97991326ce)

##### Then, select “Install suggested plugins”.

![image](https://github.com/user-attachments/assets/b14c81bd-f5f2-4023-86d8-9cb62ce372cf)

##### It will take a while, but it will install everything you need.

##### Next it will ask you to create the first admin user: assign a username, a password, a full name and your e-mail address.

![image](https://github.com/user-attachments/assets/8a3a3428-61bf-40c0-a2f9-b5f59acb89d0)

##### Then click on “Save and Continue”.

##### Now it will show the Jenkins URL, the field is already filled in so we click on “Save and Finish” and we are done!

![image](https://github.com/user-attachments/assets/00b983bd-392a-49fe-8736-626692b10a82)

![image](https://github.com/user-attachments/assets/51cfb26c-4130-4a3e-ba6b-e5d6f42222e7)

##### The next thing is to add our GitHub credentials to the Jenkins server. So click on “Manage Jenkins” on the left. Then select “Manage Credentials” under Security:

![image](https://github.com/user-attachments/assets/73100f0f-13f6-4f80-859d-33f3c7d46ad3)

##### Then select “(global)”:

![image](https://github.com/user-attachments/assets/6b0cb349-2e75-4d3b-b3f3-bc82f46570d2)

##### Then click on the button “Add Credentials”

![image](https://github.com/user-attachments/assets/9e42c002-1d6f-4fd4-bae3-bb372289ae52)

##### In this form you need to select “Username with password” in the kind field and insert your GitHub username and password and give a random ID that can be your username as well.

![image](https://github.com/user-attachments/assets/0ae75581-4691-4aa2-8b31-786afb5b43ec)


##### Then we want to allow Jenkins to access our AWS environment, so we need to add the credentials to our AWS account, in particular the AWS Access Key and AWS Secret Key. If you don’t have them, follow this guide to generate them easily:

### 1. Accessing IAM in AWS Console

##### 1. Open your web browser and navigate to the AWS Management Console.

##### 2. Sign in with your AWS account credentials.

##### 3. In the console, locate and select “IAM” (Identity and Access Management) from the services menu.

![image](https://github.com/user-attachments/assets/15eb9699-e756-466d-88aa-518921b0ed85)

### 2. Navigate to User Details

##### 1. In the IAM dashboard, select “Users” from the left navigation pane.

##### 2. Choose the IAM user for which you want to generate access keys.

![image](https://github.com/user-attachments/assets/2e3ad7e5-60ee-4ab2-aa4e-1dc5b3954364)

### 3. Generate Access Keys

##### 1. Within the selected IAM user’s details page, navigate to the “Security credentials” tab.

![image](https://github.com/user-attachments/assets/f76bd4ab-1dc5-4ccc-993c-9cf7a8075553)

##### 2. In the “Access keys” section, click on the “Create access key” button.

![image](https://github.com/user-attachments/assets/6f2bf9ba-c7ea-4375-a98a-a46b761912e8)

##### 3. Then select the Command Line Interface (CLI) and tick the confirmation for proceed to create an access key.

![image](https://github.com/user-attachments/assets/147ce77e-cbae-4da1-ba29-67c480161156)

##### Then confirm and click “Next.”

![image](https://github.com/user-attachments/assets/576e5bfb-2ff6-45ca-9fee-c4a974af8ee0)

### 4. Creating Access Key Pair:

##### Click on the “Create access key” button.

![image](https://github.com/user-attachments/assets/4f8e979b-803b-401b-81b9-d0ffed3801c2)

##### A pop-up window will appear, presenting the Access Key ID and Secret Access Key. Note that the Secret Access Key is only displayed once, so ensure you capture it securely.

![image](https://github.com/user-attachments/assets/640cdb47-e667-4759-82ad-efafcbd04f36)

### 5. Understanding Access Key Pairs

##### 1. Access Key Pairs: Think of access keys as a pair of digital keys — one called the Access Key ID and the other the Secret Access Key.

##### 2. Access Key ID — The Identifier: The Access Key ID is like a user’s name. It identifies your IAM user when they interact with AWS services. It’s public, similar to sharing your name.

##### 3. Secret Access Key — The Secret Code: The Secret Access Key is a confidential code known only to your IAM user and AWS. It’s like a secret password that your user uses to securely communicate with AWS services, ensuring the information remains private.

##### In essence, these keys work together to ensure secure and authorized communication between your IAM user and AWS, much like using a username and password for online accounts. They’re the essential tools your IAM user needs to navigate and interact with the AWS environment programmatically.

### 6. Secure Storage and Handling:

##### 1. Secure Storage After Generation: After you’ve generated your access key pair, it’s crucial to store it securely. Imagine it as safeguarding a valuable possession.

##### 2.Utmost Confidentiality for Secret Access Key: The Secret Access Key is a secret code, much like a password. Treat it with the highest level of confidentiality, just like guarding your most sensitive information.

##### 3. AWS Doesn’t Keep Secrets: AWS doesn’t keep a copy of your Secret Access Key after creation. Therefore, it’s your responsibility to ensure its safekeeping.

##### In summary, treat your access keys like precious keys to a secure vault. Keep them confidential and safe, as their security is in your hands.

### 7. Integration with AWS CLI and SDKs

#### 1.AWS CLI Configuration:

##### For AWS CLI Users: Set up the AWS Command Line Interface (CLI) on your computer.

##### Open your terminal or command prompt.

##### Use the command aws configure and input the generated Access Key ID and Secret Access Key when prompted.

##### Follow the on-screen instructions to set your default region and preferred output format.

#### 2. SDK Integration:

##### For Developers: Integrate the access keys into AWS Software Development Kits (SDKs) based on your preferred programming language.

##### Use the provided Access Key ID and Secret Access Key as credentials in your code.

##### Each SDK has its own methods for incorporating these keys; consult the SDK documentation for your language of choice.

##### In essence, configuring the AWS CLI allows you to interact with AWS services via the command line, while integrating access keys into SDKs empowers your applications to communicate with AWS programmatically using your IAM user’s credentials. These steps ensure a seamless bridge between your local environment and the vast capabilities of AWS.



