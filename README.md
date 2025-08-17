# Launching Website

The purpose of this project is to setup a HTTPS webserver on an Amazon EC2 instance using NGINX - which can be easily accessible from the browser.

## Purchasing domain via CloudFlare

The domain mazharulislam.dev is purchased through CloudFlare - admittedtly, choosing a name was very difficult...

<img width="1068" height="450" alt="image" src="https://github.com/user-attachments/assets/3a55ffd9-7183-42a9-988f-46b87b432c42" />

## Setting up Ubuntu EC2 instance

I set up an EC2 instance using Ubuntu with the following:

### Quick Start:
AMI: Default - Ubuntu Server 24.04 LTS (HVM), SSD Volume Type (Free tier eligible)
Architecture: Default - 64-bit (x86)
Instance type: Default - t3.micro
Key pair login - Yes

### Security group:
✅ Allow SSH traffic from - [My IP]
✅ Allow HTTPS traffic from the internet
✅ Allow HTTP traffic from the internet

The instance is launched.

Since I work from Windows - I launched Windows Subsystem for Linux (WSL) and used the ssh login option - making sure I `cd` into the folder my private key is located:

    ssh -i "private_key_filename" ubuntu@ec2-ip-address.eu-north-1.compute.amazonaws.com

The connection to the EC2 instance from WSL is successful.

## Setting up webserver

