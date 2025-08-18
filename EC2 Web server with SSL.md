# EC2 Webserver with SSL

The purpose of this project is to setup a HTTPS webserver on an Amazon EC2 instance using NGINX - which can be easily accessible from the browser.

This will display my own personal website mazharulislam.dev on any browser - the final output looks like this:

<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/1b734693-a703-41f8-bddf-2df7aeb10042" />


## Purchasing domain via CloudFlare

The domain mazharulislam.dev is purchased through CloudFlare - admittedtly, choosing a name was very difficult...

<img width="1068" height="450" alt="image" src="https://github.com/user-attachments/assets/3a55ffd9-7183-42a9-988f-46b87b432c42" />

## Setting up Ubuntu EC2 instance

I set up an EC2 instance using Ubuntu with the following:

### Quick Start:
AMI: Default - Ubuntu Server 24.04 LTS (HVM), SSD Volume Type (Free tier eligible)
<br>
Architecture: Default - 64-bit (x86)
<br>
Instance type: Default - t3.micro
<br>
Key pair login - Yes

### Security group:
✅ Allow SSH traffic from - [My IP]
<br>
✅ Allow HTTPS traffic from the internet
<br>
✅ Allow HTTP traffic from the internet
<br><br>
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

[This](https://www.youtube.com/watch?v=n7vKxkMIBM0&list=WL&index=2) video explains the steps to setting up nginx on Ubuntu - which are explained below.

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

## Encryption via SSL

To encrypt the HTTPS connection a library called Certbot is used. [This](https://www.youtube.com/watch?v=cBh6yTH-XY4&list=WL&index=1) video which explains how to add SSL to an nginx webserver.

The first step is to install certbot and python3 - answer "yes" or "y" for continuing installation.

    $ sudo apt install certbot python3-certbot-nginx                                       
    Reading package lists... Done                                                                                  
    Building dependency tree... Done
    ...
    Need to get 1097 kB of archives.                                                                               
    After this operation, 5699 kB of additional disk space will be used.                                           
    Do you want to continue? [Y/n] y
    Get:1 http://eu-north-1.ec2.archive.ubuntu.com/ubuntu noble/universe amd64 python3-josepy all 1.14.0-1 [22.1 kB]                                                                                                     Get:2 http://eu-north-1.ec2.archive.ubuntu.com/ubuntu noble/universe amd64 python3-rfc3339 all 1.1-4 [6744 B]
    ...
    No user sessions are running outdated binaries.                                                                                                                                                                      
    No VM guests are running outdated hypervisor (qemu) binaries on this host.

Now that the certbot is installed - `vim` is used inside the nginx.config file to set the `default_server` variable to mazharulislam.dev.

    $ sudo vim /etc/nginx/sites-available/default

Inside vim:

<img width="1051" height="409" alt="image" src="https://github.com/user-attachments/assets/0745424f-1321-4095-9b86-4187c9560cf6" />

After saving and exiting test the syntax with `nginx -t` command:

    $ sudo nginx -t                                                                        
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    
The last step is obtaining an SSL certificate from certbot - ensure your email is entered and "yes" is answered for additional permissions and acknowledgements:

    $ sudo certbot --nginx -d mazharulislam.dev                                            
    Saving debug log to /var/log/letsencrypt/letsencrypt.log                                                       
    Enter email address (used for urgent renewal and security notices)                                              
    (Enter 'c' to cancel): [INSERT EMAIL ADDRESS]                                                                                                                                                              - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -                                
    Please read the Terms of Service at                                                                            
    https://letsencrypt.org/documents/LE-SA-v1.5-February-24-2025.pdf. You must                                    
    agree in order to register with the ACME server. Do you agree?                                                 
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    (Y)es/(N)o: y
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -                                
    Would you be willing, once your first certificate is successfully issued, to                                   
    share your email address with the Electronic Frontier Foundation, a founding                                   
    partner of the Let's Encrypt project and the non-profit organization that                                      
    develops Certbot? We'd like to send you email about our work encrypting the web,                               
    EFF news, campaigns, and ways to support digital freedom.                                                      
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -                                
    (Y)es/(N)o: y    

    Account registered.                                                                                            
    Requesting a certificate for mazharulislam.dev                                                                                                                                                                                
    Successfully received certificate.                                                                             
    Certificate is saved at: /etc/letsencrypt/live/mazharulislam.dev/fullchain.pem                                 
    Key is saved at:         /etc/letsencrypt/live/mazharulislam.dev/privkey.pem                                   
    This certificate expires on 2025-11-15.                                                                        
    These files will be updated when the certificate renews.                                                       
    Certbot has set up a scheduled task to automatically renew this certificate in the background.                                                                                                                                
    Deploying certificate                                                                                          
    Successfully deployed certificate for mazharulislam.dev to /etc/nginx/sites-enabled/default                    
    Congratulations! You have successfully enabled HTTPS on https://mazharulislam.dev

The last line shows HTTPS is successfully enabled for mazharulislam.dev

Going back to the config file for nginx you can see nginx is listening for packets from port 80 - HTTP and redirecting to HTTPS:

<img width="811" height="400" alt="image" src="https://github.com/user-attachments/assets/6debe4f4-0fcf-4f18-936b-095b73b7ed94" />

This is the desired outcome.

## Final testing

The final website looks like this:

<img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/f8729ffa-f3e1-45a7-bb45-57bdb3c2836d" />

The project is complete!

### Notes

*Due to the free usage limit of EC2 I stop the instance whenever it is not in use
*Additionally, due to the nature of IPv4 addressing in EC2, every time an EC2 instance is started again, a new IPv4 address is assigned - this has to be manually updated on CloudFlare's A record each time.
