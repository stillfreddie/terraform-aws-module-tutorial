How to Build and Test a Basic Terraform Module on AWS — A Beginner-Friendly Guide

Introduction

Terraform is amazing for managing cloud infrastructure as code — but writing everything in one big .tf file? That gets messy fast. 
This is where modules come in.

Terraform Modules let you write clean, reusable, and organized code. Think of them like mini blueprints for your infrastructure, so  once you build one, you can plug it in anywhere.
 And in this step by step guide, I’ll walk us through how to build and test a simple Terraform module on AWS.

Have you’ve ever wondered:

What is a Terraform module, really?

How do I structure one properly?

How do I test it locally before going big?

You’re in the right place. This guide is perfect for beginners who want to write cleaner Terraform and deploy AWS resources the smart way.

Let’s get kicking.

Prerequisite
*********** 

We will be working with our Ubuntu Virtual Machine installed on our Oracle Virtual Box. Please Ensure you have AWS CLI installed on your linux VM and also ensure we have terraform installed on our Linux VM as well, without these 2 resources , we cannot move ahead.


Step 1.

We run this command "terraform version" to check the version of the Terraform on our Linux environment, then also run the command " aws --version " to check the version of the aws cli on our Linux environment , then proceed by creating a directory with command  "mkdir terraform_project " where terraform_project is our new folder's name, then cd into the folder with the command "cd terraform_project " then next is to create a custom directory called modules inside our main directory called " terraform_project " and a directory inside modules and call it vpc so we run the command " mkdir -p modules/vpc " once this is created, next thing we want to do is to switch to the vpc directory we just created command is we cd into the modules directory with "cd modules" then cd into the vpc directory with command "cd vpc"


Step 2.

Once we are already inside the vpc directory , then we will now write our Terraform VPC Module Code  , we will have to first create a main.tf file using the command "vim main.tf " then input these codes in the main.tf file

provider "aws" {
  region = var.region
}

resource "aws_vpc" "this" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "this" {
  vpc_id     = aws_vpc.this.id
  cidr_block = "10.0.1.0/24"
}

data "aws_ssm_parameter" "this" {
  name = "/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2"
}

After entering the codes in there, Press Escape and enter :wq to save and exit the file.


Step 3.

We will now create a new file called variables.tf with command " vim variables.tf "
 Then paste the follow codes inside the variables.tf file.

variable "region" {
  type    = string
  default = "us-east-1"
}

Press Escape and enter :wq to save and exit the file.


Step 4.

Create a new file called outputs.tf with the command vim "outputs.tf "

In the file, insert and review the provided code

output "subnet_id" {
  value = aws_subnet.this.id
}

output "ami_id" {
  value = data.aws_ssm_parameter.this.value
}


Please Note that  The code in outputs.tf is critical to exporting values to your main Terraform code, where you'll be referencing this module. Specifically, it returns the subnet and AMI IDs for your EC2 instance.
then Press Escape and enter :wq to save and exit the file



Step 5.

this step we are going to Write our Main Terraform Project Code, so all we have to do now is switch to our main project directory which is " terraform_project " command to do that is " cd ~/terraform_project "  then we create a new file called  "main.tf" with command " vim main.tf " In the file, insert and review the provided code:

variable "main_region" {
  type    = string
  default = "us-east-1"
}

provider "aws" {
  region = var.main_region
}

module "vpc" {
  source = "./modules/vpc"
  region = var.main_region
}

resource "aws_instance" "my-instance" {
  ami           = module.vpc.ami_id
  subnet_id     = module.vpc.subnet_id
  instance_type = "t2.micro"
}


Note, This code in main.tf invokes the VPC module that you created earlier. Notice how you're referencing the code using the source option within the module block to let Terraform know where the module code resides.
next thing is to Press Escape and enter :wq to save and exit the file.




Step 6. 

Create a new file called " outputs.tf "  command to create is " vim outputs.tf "

In the file, insert and review the provided code:


output "PrivateIP" {
  description = "Private IP of EC2 instance"
  value       = aws_instance.my-instance.private_ip
}
Then Press Escape and enter :wq to save and exit the file




Step 7.

This step what we are doing is to Deploy our Code and Test Out our Module, we have to Format the code in all of our files in preparation for deployment. so command to run is '' terraform fmt -recursive ", then the next stage is to generate AWS Access Key and Secret Access Key on our AWS console, after generating , we run this command " AWS configure " then it will prompt for our AWS access key , we paste it and press enter, then next it will prompt for our AWS secret access key, we copy and paste it press enter, it will prompt for our default region name we will enter us-east-1 and press enter , for default output leave it at none and press enter .




Step 8.

We are still deploying our code and testing out our module, what we have to do next is run the command " terraform init " Initialize the Terraform configuration to fetch any required providers and get the code being referenced in the module block 
 Next we run the command " terraform validate " this command  Validate the code to look for any errors in syntax, parameters, or attributes within Terraform resources that may prevent it from deploying correctly. after running this command we should receive a notification that the configuration is valid,  Next command to run is "terraform plan " this command  is like a dry run or a preview of what Terraform is about to do before it actually does it, So in short, terraform plan lets you see the changes before applying them, giving you a chance to catch mistakes or confirm the outcome before running terraform apply.

Step 9.


Then To complete the deployment we will run the command " terraform apply --auto-approve " 
Please Note: The --auto-approve flag will prevent Terraform from prompting you to enter yes explicitly before it deploys the code.
 Once the code has executed successfully, note in the output that 3 resources have been created and the private IP address of the EC2 instance is returned as was configured in the outputs.tf file in your main project code.


Step 10.



Then this command " terraform state list " will View all of the resources that Terraform has created and is now tracking in the state file.
The list of resources should include your EC2 instance, which was configured and created by the main Terraform code, and 3 resources with module.vpc in the name, which were configured and created via the module code.


Finally, we head to AWS Console, then search for Instance, And view Instances, we will see our Ec2 Instance that was created.


Please Note: if We want to tear down all the resources we have created on AWS to avoid being billed , we ll run the command terraform destroy, When prompted, type yes and press Enter.


Conclusion

And just like that, you've built your first reusable Terraform module and tested it on AWS, big win! 

By modularizing your infrastructure code, you’ve taken a huge step toward writing cleaner, more maintainable, and scalable Terraform projects. Instead of repeating the same blocks over and over, you now have a blueprint you can plug into any environment with just a few lines of code.

 Key takeaways:
You learned how to structure a Terraform module (main.tf, variables.tf, and outputs.tf)

You created a simple, testable AWS VPC resource

You used the module in a real Terraform configuration and saw it come to life on AWS

Modules are the building blocks of serious Terraform workflows — and now you know how to build your own.






