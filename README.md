# -AWS-Multi-Tier-High-Availability-Web-Architecture

<img width="720" height="274" alt="image" src="https://github.com/user-attachments/assets/ef5beaeb-1123-40dd-ae0e-14ae09754e15" />

1. Domain and DNS Setup
* Purchased a custom domain via GoDaddy.
* Configured nameservers to point to AWS Route 53.
* Created a hosted zone in Route 53.

<img width="720" height="408" alt="image" src="https://github.com/user-attachments/assets/44d9a775-b6a0-4c59-9bad-42d7eaf32f11" />

2. CloudFront and Certificate
* Created a CloudFront distribution .
* Issued an certificate via AWS Certificate Manager.
* Connected the certificate to CloudFront for secure HTTPS access.

<img width="720" height="369" alt="image" src="https://github.com/user-attachments/assets/ed37e783-696f-4fff-99bc-c6fd538d83f6" />
Created an A Record in Route 53 to map the domain to the CloudFront distribution
<img width="720" height="393" alt="image" src="https://github.com/user-attachments/assets/b3aafaae-256b-410b-b5b8-14ab6bc00710" />

3. VPC and Architecture Setup
Created a NAT Gateway in Public Subnet 1 to allow instances in private subnets to access the internet for updates and package downloads.
* 2 Public Subnets for NAT gateway and instances
* 2 Private Subnets for autoscaling (NGNIX proxy)
* 2 Private Subnets for second Load Balancer
* 2 Private Subnets for the backend application ,RDS and autoscaling

  <img width="720" height="1555" alt="image" src="https://github.com/user-attachments/assets/c5e05e48-e590-4813-9904-b7952d5c5e6a" />

  4. NGINX Reverse Proxy Setup
Launched EC2 instance in a public subnet.
Installed NGINX to forward traffic to app instances in private subnets.
Used as a reverse proxy for routing and load balancing.
sudo apt update -y
sudo apt install nginx -y

Deleted the default nginx.html file and replaced the config in /etc/nginx/nginx.conf with:

http {
upstream backend {
server <INTERNAL-LOAD-BALANCER-DNS>:80; # Load balancer forwarding to container
}

server {
listen 80;
server_name _;

location / {
proxy_pass http://backend;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
}
}

Press enter or click to view image in full size
<img width="720" height="411" alt="image" src="https://github.com/user-attachments/assets/1bd04031-2c54-458d-ac70-4042cb3b4f3e" />
<img width="720" height="370" alt="image" src="https://github.com/user-attachments/assets/9efcfd3c-a0f0-4859-852d-938a808c0590" />

5. First Load Balancer and Auto Scaling (Internet-facing )
* Created a Application Load Balancer in Public Subnet 1 and 2
* This ALB distributes traffic across Private Subnet 1 and 2 where application instances are running.
* Configured EC2 instances in target groups and set Auto Scaling policies.
* Set up NGINX as a Reverse Proxy on a separate EC2 instance.
* In Private Subnet 1 and 2, Auto Scaling was configured for NGINX ec2-instance
* The reverse proxy forwards traffic to a second Load Balancer deployed in Private Subnet 3 and 4.

  <img width="720" height="383" alt="image" src="https://github.com/user-attachments/assets/d9aa4cc7-2f16-4ccc-9b7f-3b108ca5dd85" />
  
  6. second Load Balancer and Auto Scaling(Internal)
This second ALB handles requests within the deeper application layer securely and efficiently.
In Private Subnet 5 and 6, Auto Scaling was configured for both the application tier and the RDS (using Multi-AZ deployment) to ensure high availability and performance.
<img width="720" height="400" alt="image" src="https://github.com/user-attachments/assets/48529199-e9bd-40f9-b6a8-5551adc25f47" />
<img width="720" height="390" alt="image" src="https://github.com/user-attachments/assets/95d00ab6-2cd5-4c24-8211-6b45d162e7e8" />

7. Backend Application EC2 in (Private Subnet)
Deployed in private subnet.
Application was pulled from DockerHub and run as a containerized Node.js app on port 3000.
Installed mysql-client to allow backend to connect with RDS
<img width="720" height="354" alt="image" src="https://github.com/user-attachments/assets/dfaed257-b001-4171-8b78-f714c08fc344" />
<img width="720" height="98" alt="image" src="https://github.com/user-attachments/assets/4d6354c8-452c-4eeb-836a-bea0ce3f1e0d" />

8. Database Setup with RDS
* Provisioned Amazon RDS with MySQL.
* Master-Replica deployment across availability zones.
* Secured with security groups and subnet group.

<img width="1280" height="596" alt="image" src="https://github.com/user-attachments/assets/f4f65b64-06fb-4d48-9f09-e357087dc483" />

9. Established connection between CloudFront and Load Balancer:
* Added a listener on port 443 (HTTPS) in the Load Balancer.
* Listener uses the SSL certificate from ACM.
* Forwarding rules send HTTPS traffic to the target group (NGINX or backend).

10. Testing and Go Live
Verified DNS, static and dynamic content access.
Connected CloudFront and Route 53 with the domain.
Ensured application runs end-to-end through NGINX → App → DB.
Press enter or click to view image in full size

<img width="720" height="378" alt="image" src="https://github.com/user-attachments/assets/d29b8861-33f6-4443-af1f-d79f29b00fc4" />
<img width="720" height="326" alt="image" src="https://github.com/user-attachments/assets/450dd276-451a-4c5d-a2ee-10ab3fcb6189" />















