9/9/25

Class 7: EC2 basics

task==building an Amazon Linux EC2 with a custom script

template==https://github.com/XDE77/bmc5/blob/main/ec2scrpit



steps

1. create a new security group (do not use default, allows resources to have access to areas inside/outside of AWS)
2. Create EC2
3. Copy public DNS make it into a usable link and post link into the chat

notes:
= Key pair connects a user's computer to the AWS EC2 via secure shell
=make sure to type http:// before any links are made
Example- http://ec2-3-17-6-2.us-east-2.compute.amazonaws.com
=tags are useful for organization as well as tracking resource usage across the organization, essential for the office.



tear down

1. instance state(click on arrow, there will be a drop down menu)
2. click on terminate(delete) instance
