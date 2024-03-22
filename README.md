# Hosting static website on AWS EC2 Instance with Docker Container

## Steps :


1. Create a **Security Group** on AWS EC2 : Allow port 80 (http) and 22 (ssh) inbound rule. Also make sure that the outbound 
   rule which will give access to the Internet is also enabled, otherwise you cannot install anything from the Internet to your instance.
2. In the navigation pane, under **Network & Security**, choose **Key Pairs**.
   Choose Create key pair. For Name, enter a descriptive name for the key pair. Amazon EC2 associates the public key with the name that you specify as the key name. For **Key pair** type, choose  RSA.
   For **Private key** file format, choose the format in which to save the private key. To save the private key in a format that can be used with OpenSSH, choose **.pem.**. Choose **Create key pair**.
3. Create an EC2 Instance using Ubuntu image leaving all settings to their default values. Also add below provision shell script in **Additional Details** tab:

```  
#!/bin/bash

sudo apt-get update -y
sudo apt-get install wget curl git vim ca-certificates gnupg lsb-release -y
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-cache policy docker-ce
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
sudo usermod -aG docker ubuntu
sudo sudo systemctl start docker
sudo sudo enable start docker
sudo wget https://github.com/docker/compose/releases/download/v2.14.0/docker-compose-linux-x86_64 -O /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```  

4. Create Dockerfile and nginx configuration files. For this purpose you can use vi visual editor. For using it just type `vi Dockerfile` to create a Dockerfile and add a content to it. To exit the editing mode just type :wq (which is for save and exit) or :q (exit without saving) or :!q (force exit)


Dockerfile

```
FROM nginx:latest
RUN apt-get update && apt-get upgrade -y
RUN apt-get install wget unzip -y
WORKDIR /usr/share/nginx/html
COPY default.conf /etc/nginx/sites-enabled/
ADD https://bootstrapmade.com/content/templatefiles/Ninestars/Ninestars.zip .
RUN unzip Ninestars.zip
RUN mv Ninestars/* .
RUN rm -rf Ninestars Ninestars.zip
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```  

5. Create nginx configuration file called **default.conf**:

```
server {
        listen 80 default_server;
        root /usr/share/nginx/html;
        index index.html;
        server_name mysite.com;
}
```  

6. Build Docker Image

```
docker build -t testimg:v1 . 
```  

7. Run the Container

```
docker run -it --rm -d -p 80:80 testimg:v1  
```

8. Copy your EC2's Public IPv4 DNS and open it in your browser. 

