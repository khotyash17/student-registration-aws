# Java Application Deployment with Reverse Proxy on AWS

## 🚀 Objective
Deploy a Java-based Student Registration Web Application on AWS with RDS MySQL, secured and accessed via a reverse proxy using open-source tools.

---

## 1. Infrastructure Setup
### 🔹 Backend EC2 (Application Server)
- Launch a Linux-based EC2 instance (Amazon Linux 2 recommended).
- Install Java and Apache Tomcat.

```bash
sudo yum update -y
sudo yum install java-1.8.0-openjdk -y
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.85/bin/apache-tomcat-9.0.85.tar.gz
tar -xzf apache-tomcat-9.0.85.tar.gz
mv apache-tomcat-9.0.85 ~/tomcat
chmod +x ~/tomcat/bin/*.sh
```
🔹 Reverse Proxy EC2 (Proxy Server)
- Launch another Linux EC2 instance.
- Install NGINX:
```
sudo amazon-linux-extras enable nginx1
sudo yum install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```
## 2. Application Deployment
- Copy student.war to ~/tomcat/webapps/ on the backend EC2.
- Start Tomcat:
```
~/tomcat/bin/startup.sh
```
- Verify it's accessible internally at http://<Backend-Private-IP>:8080/student
## 3. Database Setup (Amazon RDS MySQL)
- Create an RDS MySQL instance.
- Set up security groups to allow MySQL (port 3306) from backend EC2.
- Create database and table:
```
create database studentdb;
use studentdb;

create table student (
    id int auto_increment primary key,
    fullname varchar(100),
    address varchar(255),
    age int,
    qual varchar(50),
    percent float,
    yop int
);
```
## 4. MySQL Connector Configuration
- Download MySQL Connector/J:
🔗 Download mysql-connector.jar

- Copy the .jar into Tomcat's lib folder:
```
cp mysql-connector-j-8.0.33.jar ~/tomcat/lib/
```
- Restart Tomcat:
```
~/tomcat/bin/shutdown.sh
~/tomcat/bin/startup.sh
```
## 5. Reverse Proxy Configuration
🔸 NGINX Reverse Proxy
Edit /etc/nginx/nginx.conf:
```
server {
    listen 80;
    server_name _;

    location /student/ {
        proxy_pass http://<Backend-Private-IP>:8080/student/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
- Restart NGINX:
```
sudo nginx -t
sudo systemctl restart nginx
```
## 6. Security Group Setup
- Reverse Proxy EC2:
  - Allow HTTP (port 80) from anywhere.
- Backend EC2:
  - Allow port 8080 only from reverse proxy's private IP.
  - Block all other inbound traffic.
 
##  Final Access
Access the application only through:
```
http://<Reverse-Proxy-Public-IP>/student
```
