# Openswan-VPC
## What is Openswan?
<a href="https://www.openswan.org/projects/openswan">Openswan</a> is a software based IPSEC VPN. 
## What is Openswan-VPC?
Openswan-VPC adds a init.d style start/stop script for connecting Openswan to an AWS Virtual Private Cloud (VPC) using the VPN gateway.

## How do I use this?
### Setup your VPC with a VPN Gateway
The first step is to create a VPC for Openswan-VPC to connect to.

1. Use the EC2 console to create a new Elastic IP. This will be need during the Customer Gateway setup.
2. Launch a new VPC using the wizard called "VPC with a Private Subnet Only and Hardware VPN Access". Use the Elastic IP as the Customer Gateway IP and select dynamic (BGP) routing.
3. Launch an Amazon Linux instance in this VPC so you have host you can ping.
4. Download VPN setup file in Generic format
5. Configure the default security group to allow IPSEC (UDP/500) and BGP (TPC/179) inbound connections for both the Virtual Private Gateway outside IP addresses.
6. Replace the following variables in the User Data below with the values in the VPN setup file

        CGW_OUTSIDE_IP=54.200.2.26
        AWS_ASN=17493
        CUSTOMER_ASN=65000
        VGW_TUNNEL1_OUTSIDE_IP=203.83.222.236
        CGW_TUNNEL1_INSIDE_IP=169.254.251.18
        VGW_TUNNEL1_INSIDE_IP=169.254.251.17
        TUNNEL1_SECRET="secret key"
        VGW_TUNNEL2_OUTSIDE_IP=203.83.222.237
        CGW_TUNNEL2_INSIDE_IP=169.254.251.22
        VGW_TUNNEL2_INSIDE_IP=169.254.251.21
        TUNNEL2_SECRET="secret key"

### Setup Openswan-VPC
Now you will create an EC2 instance to ask the Customer Gateway and install Openswan on this instance.

1. Launch a new Amazon Linux instance from the EC2 console where you created the Elastic IP in the previous step. Edit the advanced options and put in the User Data. Select default security group and uncheck the box "auto assign public IP" since you will be using the Elastic IP as the public IP.
2. Configure the default security group to allow inbound connections to UPD/500 and TCP/179 from the Outside IP Addresses of the Virtual Private Gateway. These values are available in the VPN configuration file.
3. After the instance launches copy the Openswan-VPC vpn-gateway file to /etc/init.d and configure it to start on bootup, sudo chkconfig --add vpn-gateway
4. Install the openswan software using sudo yum install openswan quagga
5. Associate the Elastic IP with the instance and then reboot the instance.

### Testing the configuration
The Openswan configuration uses an network namespace to isolate Openswan configuration from the default network settings. 
To test, do the following.

1. ssh to Linux instance running Openswan
2. change to root and start a bash shell inside the openswan network namespace

        sudo su
        alias ox='ip netns exec openswan'
        ox bash

3. check that you are receiving routes for your VPC

        route -n 

4. ping your Linux instance inside the VPC

        ping <Linux instance inside VPC>
