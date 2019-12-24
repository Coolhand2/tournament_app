# Tournament Application
The big repository that combines all the services into one big stack that can be re-created in an AWS account.

**_This is a design document, and may be ahead of where the cloudformation stack files are._**

## Current Design

* See vpc-stack.yml

We're currently attempting to build a simple VPC with an onion of security layers.

### Subnet Design

There will be eventually four subnets, one for each AZ in us-west-2. Currently the stack is only listing 3.

**Public Subnets:**

* 10.0.0.0/24 (us-west-2a) _Name: Tournament Public A_
* 10.0.1.0/24 (us-west-2b) _Name: Tournament Public B_
* 10.0.2.0/24 (us-west-2c) _Name: Tournament Public C_
* 10.0.3.0/24 (us-west-2d) _Name: Tournament Public D_

**Private Subnets:**

* 10.1.0.0/16 (us-west-2a) _Name: Tournament Private A_
* 10.2.0.0/16 (us-west-2b) _Name: Tournament Private B_
* 10.3.0.0/16 (us-west-2c) _Name: Tournament Private C_
* 10.4.0.0/16 (us-west-2d) _Name: Tournament Private D_

### Security Layers

#### NACLs

##### Outbound Connections

We currently have no need to restrict outbound connections.

_We will be revisiting this to examine ports/protocols of outbound-container communication and lock it down._

* allow out 0.0.0.0/0

##### Inbound Connections

The UI will be served from an S3 bucket, which removes our need to open up port 80/443 for the frontend. We will be requiring SSL Client-Cert verification for API calls however, which still requires us to open port 443. We also wish to ensure all public/private subnet is explicitly allowed through.

_We will be revisiting this to examine ports/protocols of inter-container communication and lock it down._

* 0.0.0.0/0:443:_TCP_
* 10.0.0.0/24:_All Ports_:_All Protocols_
* 10.0.1.0/24:_All Ports_:_All Protocols_
* 10.0.2.0/24:_All Ports_:_All Protocols_
* 10.0.3.0/24:_All Ports_:_All Protocols_
* 10.1.0.0/16:_All Ports_:_All Protocols_
* 10.2.0.0/16:_All Ports_:_All Protocols_
* 10.3.0.0/16:_All Ports_:_All Protocols_
* 10.4.0.0/16:_All Ports_:_All Protocols_

#### Security Groups

A design choice has to be made here. Do we want individual security groups for every instance we launch, or do we wish to build generic ones that can be applied in groups, based on functionality needs?

As of right now, allowing containers to communicate with eachother as necessary for heartbeat and others, we will have at least a SG that allows for that.

_We will be revisiting this to examine ports/protocols of inter-container communication, categorize them into Securiy Groups, and lock down the VPC._

* _All Protocols_:_All Ports_:_This Security Group_ Tournament Default Group
* _TCP_:_3306_:_10.0.0.0/8_ Potential Future MySQL only security group.
* _TCP_:_443_:_10.0.0.0/8_ Potential Future SSL only security group.
* _TCP_:_11211_:_10.0.0.0/8_ Potential Future Memcached only security group.

## Todo List

-[] VPC with Subnets
-[] Route53 info
-[] IAM Users Stack for future Database considerations
-[] buildspec.yml to run all the submodule stacks.
-[] Examine Outbound API ports and connections for NACLs.
-[] Examine inter-subnet communication usage for NACLs.
-[] Examine api container communications for SGs.
