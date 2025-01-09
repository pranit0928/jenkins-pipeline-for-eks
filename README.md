## Step 1: create a keypair

##### The first thing you have to do is to create a new keypair. So in the AWS management console go to “EC2” and select “Key pairs” in the listed overview of your resources:

![image](https://github.com/user-attachments/assets/1c68ddc9-7826-40c7-83bb-d93e074d0934)

##### Then select “Create key pair” on the top-right corner.

![image](https://github.com/user-attachments/assets/a58dc44d-da0e-482c-9d1d-6c1cf1e2e5dc)

##### Give a name to the keypair and select as key file format “.pem”, then select “Create key pair” and save it in a new folder for this project, if you haven’t created it yet.

## Step 2: create the projects folders

##### In your project folder you can follow the structure of the code of my repository, you can also directly fork the repository if you prefer

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

##### Once you have the access keys, we are now ready to add them in Jenkins but we can do it only one at a time, so first of all we want to specify our AWS Access Key. So click again on “Add Credentials” and as kind select “Secret text”. It will ask you for a Secret and an ID. In ID specify “AWS_ACCESS_KEY_ID” and in Secret paste your AWS Access Key.

![image](https://github.com/user-attachments/assets/bb1924f8-1a0f-48dd-b1b4-9404d1093e6a)

##### Once you’re done, click again on “Add Credentials” and do the same for your AWS Secret Key. In ID type “AWS_SECRET_ACCESS_KEY”:

Once you’re done, click again on “Add Credentials” and do the same for your AWS Secret Key. In ID type “AWS_SECRET_ACCESS_KEY”:

![image](https://github.com/user-attachments/assets/c3b73354-3149-4b6e-b91f-c189d8a469f6)

##### Now all the credentials are set up:

![image](https://github.com/user-attachments/assets/7801e287-8902-49f8-851e-5f160f3a09b4)

## Step 4: Jenkins pipeline

##### Here I will explain the other folder “2-terraform-eks-deployment”, that we have not discussed about yet and also the folder “kubernetes”.

##### We want to create an Elastic Kubernetes Service (EKS) cluster and deploy Kubernetes manifest files inside it using Jenkins.

##### First we start from “2-terraform-eks-deployment”.

##### In “backend.tf” we configure the same backend as before, it just changes the key

##### In “eks.tf” we are defining a module to create an EKS cluster. We are using the same VPC we created earlier. The source is a module published in the Terraform registry, if you are curious and want to see what it does go and check it out on:

##### It is widely used and it will basically create several resources for us, such as EKS nodes and a Load Balancer. This one is important because we will be able to access to our application through the Load Balancer DNS name.

##### In “vpc.tf” we specify the VPC with a module but again we take the code from a module of the Terraform registry, so we don’t create it from scratch

##### And then in “variables.tf” and “terraform.tfvars” we specify the ranges of IP addresses that the VPC and the subnets can take

##### In the root folder there is a Jenkinsfile written in Groovy. This is the main file that Jenkins will use to do the deployment process for us in stages. So we will not need to deploy ourselves with the usual Terraform commands, but the Jenkins server will do that for us, as you can also see from the Jenkinsfile.

##### As you can see, we have two stages.

##### In the first stage, it initializes and applies a Terraform configuration to create an EKS cluster.

##### In the second stage, it updates the kubeconfig with the EKS cluster information and applies Kubernetes YAML files (nginx-deployment.yaml and nginx-service.yaml) to deploy Nginx to the EKS cluster. These two files are located in the “kubernetes” folder.

##### So, in nginx-deployment.yamlit is described a Kubernetes deployment for deploying a simple Nginx web server

##### In nginx-service.yaml instead we are setting up a Kubernetes Service for exposing the Nginx deployment

##### **Again:** we do not have to run the usual Terraform commands to deploy this code, because Jenkins will do that for us. We will host this code in a GitHub repository and then Jenkins will connect to it and pull it down following the deployment process described in the Jenkinsfile.

## Step 5: Create a GitHub repository

##### So Jenkins need to access to this code to do the deployment for us, so create a new GitHub repository and name it “my-jenkins-pipeline” or call it whatever you want. Leave it public and do not add a README.md file.

##### Then pull down the repository with “git clone” and move the following files to the cloning folder:

##### Jenkinsfile

##### The folder “2-terraform-eks-deployment”

##### The folder “kubernetes”

#####Then push your changes to the main branch

## Step 6: create a Jenkins pipeline

#####On the Jenkins homepage, click on “New Item” on the left.

![image](https://github.com/user-attachments/assets/e888d29f-0212-4790-a807-e852d74c8b81)

##### Then select “Pipeline” and give it a name, I call it jenkins-pipeline.

![image](https://github.com/user-attachments/assets/ef80f5ba-2cf1-41d9-9d7c-1e83e40d2a01)

##### Click OK and then scroll to the bottom of the page and under Pipeline select “Pipeline script from SCM” and as SCM choose “Git” and then you want to give your GitHub Repository URL and select the credentials we defined before.

![image](https://github.com/user-attachments/assets/a8cd6ec9-c212-44f6-9176-52e25dcf63cc)

##### In branch, specify */main instead of */master.

![image](https://github.com/user-attachments/assets/f08b02c2-a2e4-42ed-95ff-7c12ac2063bf)

##### Then click on Save.

##### Now that it is all set it up we just need to run it by clicking on “Build Now” on the left.

![image](https://github.com/user-attachments/assets/8ba333c7-b054-4b41-a9d3-2bed6e8cec11)

##### You will see the process of creation and deployment defined with the two stages that we have defined before in the code. Jenkins will take about 10–15 minutes to create the EKS cluster.

![image](https://github.com/user-attachments/assets/bd36ad5f-5ac1-4328-a2af-2866e72213c5)

##### In the bottom-left in “Build History” you should see your build running, if you click on it and then click on “Console output” in the menu on the left you should see the logs and you can see that it is creating several resources, and you can follow all the process.

![image](https://github.com/user-attachments/assets/917369c7-4455-485b-b4f0-81f3b2c74142)

![image](https://github.com/user-attachments/assets/af9f398e-a59e-473c-8fd6-fbba343b2ef3)

![image](https://github.com/user-attachments/assets/d738faac-9e47-4795-accc-4c2d134e0818)

##### Now wait until the whole deployment process is finished. 

## Step 7: Test the application

##### Alright! Now it’s time to test the application. In the AWS management console go to the “Elastic Kubernetes Service” and you should see that the cluster is running:

![image](https://github.com/user-attachments/assets/183f48d5-efd2-4bf5-9f22-26c6b528b93f)

##### If you go to the tab “Resources” you can see what resources it has created (Pods, Deployment, ..).

![image](https://github.com/user-attachments/assets/3bd7721d-f3e9-4699-b431-5e0709217002)

![image](https://github.com/user-attachments/assets/de470017-2833-4b51-b718-dd3356fdeca8)

##### If you don’t see them, there should be a message above that says that your user doesn’t have the permissions to get Kubernetes objects. Click on “Learn more” and follow the guide to grant the access to your resources. Anyway, this is just to see what happens behind the scenes.

##### Now, if you go to the “EC2” AWS service and select “Load balancers” you should see the Application Load Balancer created by Kubernetes:

![image](https://github.com/user-attachments/assets/93e5a1c6-8741-4c81-9270-3f677922f6fe)

##### If you click on it you can see its DNS name:

![image](https://github.com/user-attachments/assets/5507947c-1cf7-442e-9e82-601990f9a542)

#####Copy it and navigate to it in a new tab and you should see the Nginx welcome page:

![image](https://github.com/user-attachments/assets/99b0205c-2847-4c07-9702-59c2bade4218)


## Step 8: Clean up the resources

##### Now I will show you how to delete all the resources.

##### Go to the “2-terraform-eks-deployment” folder in your project and run:

terraform init

terraform destroy -auto-approve

##### Now go to the “1-terraform-jenkins-server” and run:

terraform destroy -auto-approve


