# DevOps Documentation for Node.js Application

## Overview
This document outlines the setup and deployment environment for a Node.js application running under PM2 on an EC2 instance. The backend is integrated with an RDS MySQL database instance and a Redis cache instance. The frontend is hosted on a separate EC2 instance without direct database dependencies.

---

## Architecture

### Components
1. **Frontend**:
   - Hosted on an EC2 instance.
   - Static files served via NGINX or Apache.
   - No direct interaction with databases.

2. **Backend**:
   - Hosted on a separate EC2 instance.
   - Node.js application running under PM2 for process management.
   - Integrated with RDS MySQL for persistent data storage.
   - Integrated with Redis for caching and session management.

### Dependencies
- **Node.js**: Ensure the appropriate Node.js version is installed.
- **PM2**: For process management.
- **NGINX/Apache**: Used for frontend file serving and reverse proxying the backend.
- **MySQL**: Hosted on AWS RDS.
- **Redis**: Hosted as a managed instance or installed on a separate EC2 instance.

---

## Backend EC2 Instance Configuration

### EC2 Instance Details
- **Instance Type**: t2.medium (or based on expected load).
- **Operating System**: Amazon Linux 2 or Ubuntu 20.04 LTS.
- **Security Groups**:
  - Allow inbound HTTP/HTTPS (ports 80, 443).
  - Allow inbound traffic on port 3000 (for Node.js).
  - Allow MySQL traffic (port 3306) from the backend instance only.
  - Allow Redis traffic (port 6379) from the backend instance only.

### Node.js Application Setup
1. **Install Dependencies**:
   ```bash
   sudo yum update -y  # For Amazon Linux
   curl -fsSL https://rpm.nodesource.com/setup_16.x | sudo bash -  # Adjust version as needed
   sudo yum install -y nodejs
   sudo npm install -g pm2
   ```

2. **Clone the Application Repository**:
   ```bash
   git clone <repository-url>
   cd <application-directory>
   npm install
   ```

3. **Environment Variables**:
   Use `.env` files or AWS Systems Manager Parameter Store to manage secrets.
   ```
   DB_HOST=<RDS-endpoint>
   DB_USER=<username>
   DB_PASSWORD=<password>
   DB_NAME=<database-name>
   REDIS_HOST=<redis-endpoint>
   REDIS_PORT=6379
   ```

4. **Start Application with PM2**:
   ```bash
   pm2 start app.js --name "backend-app" --env production
   pm2 startup
   pm2 save
   ```

### RDS MySQL Setup
- **Instance Type**: db.t2.micro (scale as needed).
- **Access**:
  - Configure inbound rules to allow traffic only from the backend EC2 instance.
  - Ensure SSL/TLS for database connections.

### Redis Setup
- **Instance Type**: Managed AWS Elasticache Redis or standalone EC2 instance.
- **Configuration**:
  - Allow connections only from backend EC2 instance.
  - Ensure proper IAM roles or Redis password authentication.

---

## Frontend EC2 Instance Configuration

### EC2 Instance Details
- **Instance Type**: t2.micro (or based on expected load).
- **Operating System**: Amazon Linux 2 or Ubuntu 20.04 LTS.
- **Security Groups**:
  - Allow inbound HTTP/HTTPS (ports 80, 443).

### Frontend Setup
1. **Install Dependencies**:
   ```bash
   sudo yum update -y
   sudo yum install -y nginx
   ```

2. **Deploy Static Files**:
   - Copy built files from the frontend repository.
   ```bash
   scp -r ./dist/ <ec2-user>@<frontend-instance-ip>:/var/www/html
   ```

3. **Configure NGINX**:
   - Update `/etc/nginx/nginx.conf` or create a site-specific configuration file:
   ```nginx
   server {
       listen 80;
       server_name <frontend-domain>;

       root /var/www/html;
       index index.html;

       location / {
           try_files $uri /index.html;
       }
   }
   ```
   - Restart NGINX:
   ```bash
   sudo systemctl restart nginx
   ```

---

## Monitoring and Maintenance

1. **Monitoring Tools**:
   - AWS CloudWatch for instance and service metrics.
   - PM2 logs and metrics for backend application.

2. **Backup**:
   - Enable automated RDS backups.
   - Use a tool like `cron` or AWS Backup for periodic Redis snapshots if not using Elasticache.

3. **Scaling**:
   - Use AWS Auto Scaling groups for both frontend and backend.
   - Leverage AWS Elastic Load Balancing (ELB) for traffic distribution.

---

## Security Best Practices
- Use IAM roles to manage permissions securely.
- Keep EC2 instance OS and packages up to date.
- Implement HTTPS using certificates (e.g., AWS Certificate Manager or Let's Encrypt).
- Restrict access to sensitive ports using security groups.
- Regularly audit application and server logs.

---

## Conclusion
This setup provides a robust and scalable environment for deploying a Node.js application with a clear separation of concerns between the frontend and backend. Adopting best practices ensures security, scalability, and ease of management.

