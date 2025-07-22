---
title: "VPC Peering with Public Subnet and Private Subnet"
datePublished: Tue Jul 22 2025 09:39:23 GMT+0000 (Coordinated Universal Time)
cuid: cmdecf0wh001e02k0he3s97dl
slug: vpc-peering-with-public-subnet-and-private-subnet
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1753176995113/f7632e78-a8b3-4f76-b745-195f10de37c7.png
tags: awsvpc

---

In this lab, there are two VPCs. VPC 1 and VPC 2. VPC 1 has one EC2 connected to public subnet. VPC 2 had one EC2 connected to private subnet. These two VPCs will be connected via VPC peering.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753176460095/21b9b643-0cdf-4551-a994-b87ac6d34ae8.png align="center")

The end result will be look like this. From EC2 1 public IP address should be able to ping to private IP address of EC2 2.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753176469368/14ebf09f-5139-44d5-b02e-d310ea1cbd00.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753176474403/86552c61-f153-457f-9257-5d54a9abb5cd.png align="center")

This is the Project VPC with CIDR range of 10.0.0.0/16

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753176481430/7570ea53-3f16-4267-954c-052a1a294012.png align="center")

This is the test VPC with CIDR range of 172.16.0.0/16

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753176488455/2e399adb-c171-4ad2-9b20-d9f7dc5c2e49.png align="center")

In VPC peering I have connected two VPCs.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753176492246/c716d164-5130-4f93-ae25-43885d8ab52b.png align="center")

In Project VPC, I have config the route tables to test VPC using VPC peering connection.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753176497755/bde4b2e7-c8af-4339-843a-251139a2d849.png align="center")

In test VPC’s private subnet, I have config the route tables to Project VPC using VPC peering connection.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753176502448/654c9c6a-327a-4839-8cdf-a611af21fe8b.png align="center")

I have used VPC Reachability Analyzer to have connectivity from Instance to Instance.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753176506636/16ce6c01-9afa-4cdb-8964-79fb13119673.png align="center")

Note. You have to include ICMP in security group to be able to ping. Or use telnet.

If there are many VPC that needed to be connected we use Transit Gateway instead of VPC Peering.