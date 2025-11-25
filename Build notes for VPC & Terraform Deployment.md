VPC & Terraform Deployment setup:

remember: terraform I(V) PAD = init, validate, plan, apply, destroy

throughout the entire process make sure that code is saved or check autosave

To make sure everything is installed correctly: Go to VSC > run curl https://raw.githubusercontent.com/aaron-dm-mcdonald/Class7-notes/refs/heads/main/101825/check.sh | bash in bash terminal (https://github.com/aaron-dm-mcdonald/Class7-notes/commit/edc5caa2db762ba89a86415e4b232c7b5948c6ec#diff-9e1f34f6d384941e7b777cb863b1981a590f9e584c6b54d43dfddad446194788)

Create VPC Arch/VPC CIDR/3 private and 3 public subnets/1 Nat gateway/enable hostnames and resolution

Go to AWS console
Create VPC(VPC and more, nametag, CIDR ex. 10.10.0.0/16, default tenancy, 3 AZ, public and private CIDR blocs end in /24, 1 AZ, no endpoints, additional tags


Open admin GitBash(run as administrator)
Go to terraform folder
Commands(directory name is custom): 
mkdir 102525
cp ../.gitignore (name of the directory just made)/
cd 102525(must be same as whatever named above)
touch 0-auth.tf 1-vpc.tf etc.
code ./
run curl https://raw.githubusercontent.com/aaron-dm-mcdonald/Class7-notes/refs/heads/main/101825/check.sh | bash in bash terminal 

Virtual Studio code
go to 0-auth.tf
go to terraform registry(https://registry.terraform.io/), browse providers, AWS, use provider, copy code and paste to 0-auth
Under provider: region = "whatever region your AWS is in" & resource provider default tags, erase environment, match tags with tags put in VPC previously created 


please remember to use # to create comments (won't affect code)
make sure to have bash terminal up in VSC (terminal > new terminal > plus sign drop down menu > select default profile > select gitbash at the top)


in bash terminal of VSC: 
terraform init(run when starting terraform in a new folder, or when adding new providers to existing code. a successful run == ".terraform" folder in the directory and ".terraform.lock.hcl file in the directory)/terraform validate(catch any code issues prior to plan/apply steps. a successful run == message saying "success! the configuration is valid.)


create 1-vpc.tf in VSC
go back to terraform registry and click on documentation
look for Resource: aws_vpc(Under VPC Virtual Private Cloud), copy basic uses with tags code to 1-vpc.tf, after "aws_subnet", name it public_a/b/c", add the cidr block (one ending in /16), tenancy is default, add enable_dns_hostnames = true & enable_dns_support = true, tags name is the name of the vpc


in bash terminal of VSC:
terraform validate/terraform plan(dry run or shakedown of currently saved terraform code in the folder. can save to output, good to do for jobs, not needed in class *yet*. a successful run == if looking at the files in VS code, you will temporarily see files containing ".tfstate". Plan output will show up in the terminal, including all the options available for the resource that you are building. "plan: # to add, # to change, # to destroy." Do not touch the state file.)/terraform apply(provision terraform code into the cloud. will provision EXACTLY AS WRITTEN. TERRAFORM IS NOT WRONG; triple check before deploying. a successful run == terraform.tfstate file in the folder that did not disappear. terminal will say "Apply complete!" "# of changes applied" or "no changes" no red text == good. terraform will ask me if I want to apply changes, and I have to type "yes" to accept changes. Do not touch the tfstate file.)


To double check, go back to aws console and check your VPCs. Check the area that you placed/created the VPC in.


create new 2-subnets.tf file in VSC
use reference from class 5/2-Subnets.tf
go back to aws console and switch to EC2, check under service health to see which AZ's to use (look at zone name), replace "main" with name with public, vpc id would be aws_vpc.main.id, cidr block would be the public subnets ending in /24, add availability_zone = "name of az", change name to "public-name of az", go to console, subnets, edit subnet settings, add map_public_ip_on_launch = true 


in bash terminal of VSC:
terraform validate/terraform plan/terraform apply -auto-approve
to make sure everything is done correctly step by step, edit-toggle line comment in VSC
On the private part of the route table, may need to change gateway_id to nat_geteway_id


proceed to create the private subnets in the same format as the public subnets, after name of "aws_subnet" name it "private_a/b/c"map_public_ip_on_launch = false, tag name would be "private-name of az"


in bash terminal of VSC:
terraform validate/terraform plan/terraform apply


create new 3-igw.tf file in VSC (for public ips)
go to documentation
Resource: aws_internet_gateway (under VPC Virtual Private Cloud)
copy example usage to the .tf
make sure "main" name matches "main" name under vpc.tf
tag name could be changed to "terraform-igw" or whatever the vpc is


in bash terminal of VSC: 
terraform validate/terraform plan/terraform apply


create a new 4-nat.tf (for private ips)
in documentation gateway look for Resource: aws_eip (under EC2 Elastic Compute Cloud)
copy allocating eip from the byoip pool, remove public pool line, change name to "name of vpc-nat-eip", add tags from igw but change name to "terraform-eip-for-nat" or "name of vpc-nat-eip"
add depends_on = [ aws_internet_gateway.igw ] (this is an explicit dependency)

look for resource: aws_nat_gateway (under VPC Virtual Priavate Cloud) and copy code for Public NAT to VSC
change example to "main"
allocation_id = aws.eip.name of vpc-nat-eip.id 
subnet_id = use same public subnet used in igw (ex: aws_subnet.public_a.id)
change tags name to "name of vpc-nat-gateway" 
change depends_on = [ aws_internet.gateway.igw ]


in bash terminal of VSC:
terraform validate/terraform plan/terraform apply


please be patient with nat gateway starting up
check nat gateways in ec2


create a new 5-rtb.tf (route tables)
in documentation look for resource: aws_route_table (under VPC)
copy the basic example code and put it into the VSC
delete ipv6 segment of route code leaving the first part and the tags, change example to public and change vpc_id to aws_vpc.name of vpc-vpc.id
change cidr_block = "0.0.0.0/0"
gateway_id = "aws_internet_gateway.igw.id"
under tags change name to "terraform-public-rtb" or "name of vpc-public-rt"
go back to documents and go to resource: aws_route_table_association
copy the subnet id code and put it under the tags code in the VSC
change "a" to a public subnet ex: "public_a"(please prefer to your previously made subnets)
change subnet_id = aws_subnet.public_a.id
change route_table_id = aws_route_table.public.id
duplicate that route table code twice in order to create public_b and public_c and replacing all with the appropriate public name


in bash terminal of VSC:
terraform validate/terraform plan/terraform apply -auto-approve
check in vpc dashboard on aws console to validate changes


copy all code that was done creating the route tables in order to create the private route table. Replace all mentions of public to private as well as changing subnet names to private


in bash terminal of VSC: 
terraform validate/terraform plan/terraform apply -auto-approve
check in vpc dashboard on aws console to validate changes
terraform destroy -auto-approve/terraform apply  -auto-approve


for teardown:
terraform destroy -auto-approve
terraform destroy (destroy provisioned terraform code within the cloud. nuclear option, but good for class. many more graceful ways to destroy infrastructure; will learn this eventually. if terraform destroying at the job, pray before doing so. a successful run == terminal text saying "destroy complete. Resources: # destroyed")


create a new 6-SG (security group)
in documentation go to aws_security group (under VPC) and copy basic usage code
to make sure everything is done correctly step by step, edit-toggle line comment in VSC
Change security group name, adjust description, make sure vpc_id is referencing the vpc that has been made
On the private part of the route table, may need to change gateway_id to nat_geteway_id

in bash terminal of VSC:
terraform validate/init/plan/apply

to uncomment, toggle line comment again (highlight code)

for the ingress rule, change name to the name of the sg and add -ssh
change id to the named security group in place of allow_tls
change port to 22 (for SSH)
to be extra fancy add tags and a name

in bash terminal of VSC: 
terraform validate/init/plan/apply

always check security group in AWS for every application in VSC

for egress group be sure to change th security group id to reference the sg that was created (you can just add the sg name in front of -allow)
to be extra fancy add tags and a name

terraform validate/init/plan/apply
check both inbound and outbound rules to make sure both come up in AWS

copy already working SSH code and edit it to make http, change port to 80

terraform validate/init/plan/apply
all ipv4, no ipv6 needed for now


create a new 7-EC2.tf (for EC2 instance)

go to documentation and look for aws_instance (under EC2 elastic compute cloud/resources)

go to Network and credit specification example but only copy all the lines starting at resource "aws_network_interface" "example" segment (not the primary network interface or credit spec)

in AWS go to EC2/instances/launch an instance
under AMI, copy the AMI ID and paste in place of ami in the ami part of the code (also match the region and instance type)
make sure to rename

add  security_groups = [aws_security_group.whatever the name-sg.name] under instance_type

add subnet_id = aws_subnet.whatever public subnet you are working in.id

add associate_public_ip_address = true

add tags and name the ec2


create a user_data.sh
go to class 5/7-launchtemplate.tf and copy code starting from  #!/bin/bash and ending at rm -f /tmp/local_ipv4 /tmp/az /tmp/macid
