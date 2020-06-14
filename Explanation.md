In this example, let me walk you through an allocation of 10.0.0.0/16 that we’ll use for a VPC. Now, lets assume we have two Availability Zones, and for fault tolerance, we’re going to make a pair of subnets, each with around 250 addresses, one in each AZ. We’ll start at the beginning of this range, assigning 10.0.0.0/24 to AZ A, and 10.0.1.0/24 to AZ B.

Sounds simple.

Now putting this into a CloudFormation template: you ended up with either the hard coded addresses in your text file (horrid, no portability to other environments)or you could use another script to generate the CloudFormation template on the fly, or typically, defined a number of Parameters for the template and ask the user to supply the broken-down subnet allocations.

Which got ugly.

If you had more than 2 AZs you were allocating across.

And multiple sets of subnets (Databases, App Servers, Internal Load Balancers, External Load Balancers)

You would end up with a large number of parameters that were just CIDR blocks.

Fn::Cidr -> comes to our rescue

 fn::Cidr, that would allow it to perform subnet calculations; splitting apart a defined CIDR block into a list of small allocations, which could then be selected from and used for the address allocation. And it works for both IPv4 subnet calculations, and IPv6. 

===

  "SubnetExtA": {
      "Type": "AWS::EC2::Subnet",
      "DependsOn": "VpcCidrBlockIpv6",
      "Properties": {
        "CidrBlock": { "Fn::Select": [ 0, { "Fn::Cidr": [ {"Ref":"VpcCidr"}, "256", "8" ] } ] },
        "Ipv6CidrBlock": { "Fn::Select": [ 0, { "Fn::Cidr": [ { "Fn::Select": [ 0, { "Fn::GetAtt": [ "VPC", "Ipv6CidrBlocks" ] } ] }, "256", "64" ] } ] },
        "AssignIpv6AddressOnCreation": "true",
        "AvailabilityZone": { "Fn::Join": [ "", [ { "Ref" : "AWS::Region" }, "a" ] ] },
        "Tags": [ { "Key": "Name", "Value": { "Fn::Join": [ "-", [ { "Ref": "AWS::StackName"}, "Ext" ] ] } } ],
        "VpcId": { "Ref": "VPC" }
      }
    },

===
 
 "CidrBlock": { 
  "Fn::Select": [ 
    0, { 
      "Fn::Cidr": [ 
        {"Ref":"VpcCidr"},
        "256",
        "8"
      ]
    }
  ] 
},

----

	

Virtual Private Cloud is a construct in AWS that gives the customer their own, er, virtual network for the deployment of network based resources such as virtual machines and more. Its been around for nearly a decade, and is a basic construct that helps provide security of those resources within an AWS Region.

CloudFormation is the (text, either YAML or JSON) templating language (service) that can take a definition of resources you would like configured, and does the execution of creating these resources for you, saving you the hassle of having to either navigate the web console for hours, or scripting up many API calls (which could be thousands of API create calls).

VPCs can be quite complex; they can specify subnets for resources, across multiple Availability Zones within a Region, define routing tables, Endpoints to create, and much more. So it probably comes as no surprise that managing a VPC via CloudFormation is a natural desire. The configuration of the virtual network for a workload needs to be as management in a CI/CD fashion as the workload that will live in there.

But there’s often been a limitation in making this simple; mathematics.
VPC CIDR Blocks

The only questions that needs to be defined when creating a VPC today is the IPv4 address allocation to use. One typically chooses a non-routable IPv4 range, from the RFC1918 set (think: 10.0.0.0/8 and likewise), that is not already defined in the ‘internal’ network of an organisation. Of course, this goes horribly wrong with Mergers and Acquisitions, where two previously separate entities suddenly come together to find they have overlapping address spaces in their merged assets (so go see the approach for IPv6 to solve this – no NAT).

In this example, let me walk you through an allocation of 10.0.0.0/16 that we’ll use for a VPC. Now, lets assume we have two Availability Zones, and for fault tolerance, we’re going to make a pair of subnets, each with around 250 addresses, one in each AZ. We’ll start at the beginning of this range, assigning 10.0.0.0/24 to AZ A, and 10.0.1.0/24 to AZ B.

Sounds simple.

Now putting this into a CloudFormation template: you ended up with either the hard coded addresses in your text file (horrid, no portability to other environments)or you could use another script to generate the CloudFormation template on the fly, or typically, defined a number of Parameters for the template and ask the user to supply the broken-down subnet allocations.

Which got ugly.

If you had more than 2 AZs you were allocating across.

And multiple sets of subnets (Databases, App Servers, Internal Load Balancers, External Load Balancers)

You would end up with a large number of parameters that were just CIDR blocks.
Introducing fn::Cidr

Recently CloudFormation added an intrinsic function, fn::Cidr, that would allow it to perform subnet calculations; splitting apart a defined CIDR block into a list of small allocations, which could then be selected from and used for the address allocation. And it works for both IPv4 subnet calculations, and IPv6. While the dust on the documentation is settling, its a little complex. First, let me show you an example:

        "SubnetExtA": {
          "Type": "AWS::EC2::Subnet",
          "DependsOn": "VpcCidrBlockIpv6",
          "Properties": {
            "CidrBlock": { "Fn::Select": [ 0, { "Fn::Cidr": [ {"Ref":"VpcCidr"}, "256", "8" ] } ] },
            "Ipv6CidrBlock": { "Fn::Select": [ 0, { "Fn::Cidr": [ { "Fn::Select": [ 0, { "Fn::GetAtt": [ "VPC", "Ipv6CidrBlocks" ] } ] }, "256", "64" ] } ] },
            "AssignIpv6AddressOnCreation": "true",
            "AvailabilityZone": { "Fn::Join": [ "", [ { "Ref" : "AWS::Region" }, "a" ] ] },
            "Tags": [ { "Key": "Name", "Value": { "Fn::Join": [ "-", [ { "Ref": "AWS::StackName"}, "Ext" ] ] } } ],
            "VpcId": { "Ref": "VPC" }
          }
        },

There’s two usages here, one for the IPv4 address allocation to the subnet, and one for IPv6.
IPv4 CidrBlock

Lets bust open the above CidrBlock:

    "CidrBlock": { 
      "Fn::Select": [ 
        0, { 
          "Fn::Cidr": [ 
            {"Ref":"VpcCidr"},
            "256",
            "8"
          ]
        }
      ] 
    },

Starting form the inside, we have a Ref to a parameter, VpcCidr. This is the one parameter we have to ask for in a VPC creation, the entire VPC’s primary IPv4 address range. In our case, this is 10.0.0.0/16.

But around that we have this Fn::Cidr happening. This is extracting 256 equally sized subnets, with a host bit mask length of 8, which is a /24, as 32 (entire address size in bits) – 8 (host mask size) = 24 (subnet size). This function is returning 256 smaller subnets, as a list, so in this first usage, we simply select the inital element (stating from 0). In our next AZ, for the next subnet, we’d chose to select the second element (1), and so on


===

Ipv6CidrBlock

You’ll note that we don’t ask the user for an IPv6 address block in AWS; that’s because the only IPv6 addresses that can be used (without building abstraction networks) are those that AWS provides. A real world /64 address handed to any customer VPC that wants one. Ideal for putting in any public facing subnets to permit dual-stack deployments for your front door, while still using IPv4 for the inside. Of course, any deployment you’re looking at doing now that you’re likely to still run in the next decade, may find that the on premise networks have go dual stack over that time, so you may as well assign IPv6 to all subnets defined; but you’d perhaps not route ::/0 (the default , all routes) to the Internet directly.

Having request an IPv6 allocation and typing it to the VPC by way of a “AWS::EC2::VPCCidrBlock” type, we simply read back that range now assigned to the VPC with “Fn::GetAtt”, select the first (0th) IPv6 range, then break that into blocks with Fn::Cidr, and against incrementally select one of the list of smaller allocations back.

====


!GetAtt

When we need to access Parameters, we use !Ref. !GetAtt provides similar functionality for accessing resource attributes in the template.

The !GetAtt documentation provides the attributes available for each resource. A VPC has a “DefaultSecurityGroup” attribute. We reference this security group using !GetAttr VPC.DefaultSecurityGroup.

====

A CloudFormation creation policy is helpful if you provision an EC2 instance, typically by using user data. By default, when CloudFormation creates and EC2 instance it will not wait for the operating system and application to be ready. With a creation policy, you can ask CloudFormation to wait for an external signal.

