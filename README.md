UAT/SIT Environment:

Instances:
o	Nginx instance
●	AMI: RHEL 8
●	EC2 instance: t2.micro
●	Nginx_internet security group attached, allowing the inbound connections from anywhere in the internet on the port 80(http), 443(https), 8081(Nginx application), 22(SSH port)
●	Subnet in us-east-2a will be attached to the this instance, tags on the subnet are “environment:UAT” and “Tier:nginx”. 
●	The route table attached to the subnet allows connectivity with in the VPC, and uses the internet gateway for routing traffic to the internet.
●	The NACL are default one allowing all inbound and outbound traffic at subnet level.
●	The server host the static files on the url like https://<public-ip-nginx instance>/*.png, to serve the static file “logo.png”. The urls https://<public-ip-nginx instance>/*.css, https://<public-ip-nginx instance>:8081/static serve the static css file “.css”
●	The nginx instance uses the https for secure connection.
●	The nginx instance acts as a reverse proxy for the backend tomcat server for serving the dynamic application, the url https://<public-ip-nginx instance>/webapp, , forwards the request to the tomcat server http://<public-ip tomcat_instance>:8080/webapp
●	The static files are copied on the server in the folder /var/www/static/
●	The nginx configuration file for static files is named “static.conf” and is placed in /etc/nginx/conf.d/, which sets the url and the listening ports and the SSL certificates.

o	Tomcat instance
●	AMI: RHEL 8
●	EC2 instance: t2.micro
●	“tomcat_internet” security group attached, allowing the inbound connections from anywhere in the internet on the port 8080(tomcat application), 22(ssh)
●	Subnet in us-east-2b will be attached to the this instance, tags on the subnet are “environment:UAT” and “Tier:tomcat”. 
●	The route table attached to the subnet allows connectivity with in the VPC, and uses the internet gateway for routing traffic to the internet.
●	The NACL are default one allowing all inbound and outbound traffic at subnet level.
●	The webapp.war is deployed in the location on the server /usr/share/tomcat/webapp



 
Architecture of UAT environment

UAT deployment:
Below are the requirement for the local machine from where you will run the ansible playbook for deployment.
1.	 Should have python installed, and boto3.
2.	Should have your AWS account usre’s AWS access key and aws secret key, it will be asked while running the Ansible playbook.
3.	Unzip the UAT.zip which have the required playbooks, it has the files folder which contains the build output “static.zip” and “webapp.war” , the templates folder contains the nginx config file for making the application url.
4.	Run the “provision.yaml” with command “ansible-playbook provision.yaml” by opening terminal in the folder.
UAT environment working:
The UAT environment works with NGINX instance as the front facing server, which serves the static files and acts as a proxy for the dynamic app deployed on the tomcat instance.
To access the static files below are the url one can use:
https://<public_ip_of_nginx instance>/static 🡪 This will serve company.css file
https://<public_ip_of_nginx instance>/*.png -> where * means anything ending with png, this will serve logo.png
https://<public_ip_of_nginx instance>/*.css -> where * means anything ending with .css, this will serve company.css
https://<public_ip_of_nginx instance>/webapp will redirect the request to the tomcat server and it will serve the dynamic application of war file.
  
  ![image](https://user-images.githubusercontent.com/26652317/130961544-1c0d66a4-298b-42d3-a19e-70b4c2981a83.png)






Production deployment:
For production deployment , to make the infra highly available and withstand a single server failure. It will have the below componenets

1.	Internet facing ELB(elastic load balancer service from AWS), whose endpoint will listen for the request on port 443 to serve the https request. The ELB will be connected to the two subnet, let suppose subnet 1 in us-east-2a and subnet 2 in us-east-2b.
2.	Behind the ELB1 , there will be 2 Nginx instances running in the subnet subnet1 and subnet 2, so that the request can be load balanced between the 2 Nginx instances. The nginx servers as they will be serving the request they need to be have high CPU and memory, so would recommend using t3.large instances for both the instances. The function of these 2 instances to get the request from the ELB and serve the files as required. If it’s a static file request, it will serve via the nginx local deployment of static files.
If the request comes for the webapp, then it will pass the request to the ELB 2 by proxying the request to ELB 2 endpoint on port 8080.
3.	ELB 2 will be a private load balancer sitting behind the nginx servers and listening the request on the port 8080. It will have 2 subnets let suppose subnet 3 in us-east-2c and subnet 4 in us-east-2d. Behind the ELB2 there will be 2 instances of tomcat servers will be running listening on the port 8080, which will serve the request forwarded by nginx server to ELB 2. The tomcat servers insatances could be t3.large , they will have the webapp.war file deployed in them,
By this we can have a highly available and responsive prod environment. Below sketch outlines the plans.

 

Prod deployement



