# EC2 Webserver with SSL

The purpose of this project is to setup a HTTPS webserver on an Amazon EC2 instance using NGINX - which can be easily accessible from the browser.

This will display my own personal website mazharulislam.dev on any browser.

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

The instance is launched - a note is made of the IPv4 address, which will be used later for mazharulislam.dev domain to point towards.

Since I work from Windows - I launched Windows Subsystem for Linux (WSL) and used the ssh login option - making sure I `cd` into the folder my private key is located:

    ssh -i "private_key_filename" ubuntu@ec2-ip-address.eu-north-1.compute.amazonaws.com

The connection to the EC2 instance from WSL is successful.

## Setting up A-record for DNS

Simply having the domain name is not enough - the ip connection has to be made to the webserver and for this CloudFlare's DNS is used.

For this navigate to the domain overview for mazharulislam.dev and under DNS > Records - an A record can be created using this domain and the IPv4 address from the EC2 instance:

<img width="1638" height="165" alt="image" src="https://github.com/user-attachments/assets/0d8690a8-5c78-43d8-8d71-6143e240893b" />

Once the webserver is up and running, this connection will be checked.


## Setting up webserver

Using `sudo apt update` and `sudo apt install nginx` the latest files for nginx are installed:

    $ sudo apt update                                                                      
    Hit:1 http://eu-north-1.ec2.archive.ubuntu.com/ubuntu noble InRelease                                          
    Get:2 http://eu-north-1.ec2.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]                         
    Get:3 http://eu-north-1.ec2.archive.ubuntu.com/ubuntu noble-backports InRelease [126 kB]    
    ...
    108 packages can be upgraded. Run 'apt list --upgradable' to see them. 
    
    $ sudo apt install nginx                                                               
    Reading package lists... Done                                                                                  
    Building dependency tree... Done 
    ...
    No VM guests are running outdated hypervisor (qemu) binaries on this host.

To check if the server ip was able to be found via DNS the `nslookup` and `dig` command were used to query this:

    $ nslookup mazharulislam.dev                                                           
    Server:         [myip]                                                                                     
    Address:        [myip]#[myport]                                                                                                                                                                                                 
    Non-authoritative answer:                                                                                      
    Name:   mazharulislam.dev                                                                                      
    Address: XX.XX.XXX.108 

    $ dig mazharulislam.dev
    ...
    ;; ANSWER SECTION:                                                                                             
    mazharulislam.dev.      136     IN      A       XX.XX.XXX.108

Both sections point towards the correct IPv4 address.

Using `systemctl` command the `nginx` webserver is checked to see if it is running.

    $ sudo systemctl status nginx                                                          
    ● nginx.service - A high performance web server and a reverse proxy server                                          
    Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)                               
    Active: active (running) since Sun 2025-08-17 16:29:46 UTC; 1h 19min ago 
    ...

Under `Active` it says `active (running)` which is desired.

Since this already happened without me starting nginx I used the `systemctl start` command and re-ran `status` in case:

    $ sudo systemctl start nginx
    $ sudo systemctl status nginx                                                          
    ● nginx.service - A high performance web server and a reverse proxy server                                          
    Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)                               
    Active: active (running) since Sun 2025-08-17 16:29:46 UTC; 1h 19min ago 
    ...

To test if nginx works - the data from the server will be retrieved with the `curl` command.

        $ curl localhost
        <!DOCTYPE html>                                                                                                
        <html>                                                                                                         
        <head>                                                                                                         
        <title>Welcome to nginx!</title>
        ...
        <p>If you see this page, the nginx web server is successfully installed and                                    
        working. Further configuration is required.</p>
        ...
        </body>                                                                                                        
        </html>

This works - which is great!

I attempted to test from the web browser of my Windows PC on Google Chrome by typing the website and searching.

I recieved a "refused to connect" error.

The site is HTTP - and not HTTPS, and found out online that most browsers do not display website content from HTTP by default, as it is unsecure, and malicious entities can steal data over this connection.

It needs to be encrypted using an SSL certificate, i.e., become HTTPS.

To do this I came across [this](https://www.youtube.com/watch?v=cBh6yTH-XY4&list=WL&index=1) video which explains how to add SSL to an nginx webserver.

