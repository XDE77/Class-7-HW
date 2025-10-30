Setup VPC
1.Search vpc in searchbar
2.Create VPC
3.VPC and More
4.Enter name tag
5.Update CIDR block
6.No IPv6 CIDR
7.3 availability zones
8.Keep 3 public subnets(for assignment)
9.Change to 6 private subnets(for assignment)
10.Customize subnets for CIDR blocks
11.Edit the IPs and make sure to change the /20 to /24. Double check work.
12.In 1 AZ for nat gateways(for assignment)
13.No S3 Gateway(for assignment)/Double check work.
14.Create VPC


Teardown VPC
1.Under virtual private cloud select nat gateways/select the gateway/actions/delete nat gateway/type delete then select delete/have to wait until fully gone to release elastic IP
2.Go to elastic IP/select elastic IP/actions/release IP addresses
3.Make sure all instances are terminated first
4.Go toyour VPCs/select VPC/type delete then delete