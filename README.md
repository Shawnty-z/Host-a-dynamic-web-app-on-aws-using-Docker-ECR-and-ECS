# Host a Dynamic Web App on AWS using Docker, ECR, and ECS

## Project Overview

This project demonstrates how to host a dynamic web application on AWS using Docker containers, Amazon Elastic Container Registry (ECR), and Amazon Elastic Container Service (ECS). The architecture ensures scalability, security, and automation while leveraging key AWS services like RDS for database management and S3 for storage.

## Architecture Overview

![Architecture Diagram](/Host a Dynamic Web App on AWS using Docker, ECR, and ECS.png)

### Key Resources Used:

- **Amazon ECS**: To run the containerized application using Fargate.
- **Amazon ECR**: For storing Docker images securely.
- **Amazon RDS**: MySQL relational database hosted in a private subnet with a standby DB for failover.
- **S3**: To store files containing environment variables and SQL migration scripts.
- **Docker**: For containerizing the application and ensuring portability.
- **Auto Scaling Group**: To dynamically create new ECS tasks based on traffic.
- **Amazon Route 53**: To handle domain name resolution for the web application.
- **Application Load Balancer**: To distribute incoming web traffic across ECS tasks.
- **IAM Role**: To grant ECS tasks the necessary permissions to interact with S3.

## Problem Solved

This project addressed the challenge of deploying a web application that can:

- **Scale dynamically**: Using ECS with Fargate allows the application to scale in response to traffic, ensuring consistent performance.
- **Maintain high availability**: Leveraging Amazon RDS with a standby DB ensures that the database is resilient to failures.
- **Secure sensitive information**: Using private subnets and IAM policies to restrict access, sensitive information like database credentials and application environment variables are protected.
- **Automate deployment**: Using Docker and a series of bash scripts, the deployment of the application is fully automated, minimizing manual intervention.

## Bash Deployment Scripts

The deployment is automated using several shell scripts:

1. **build_image.sh**:
   This script builds the Docker image using provided build arguments, including the web files, GitHub credentials, and database connection details. The built image is then pushed to Amazon ECR.
   ```bash
   docker build \
   --build-arg PERSONAL_ACCESS_TOKEN= \
   --build-arg GITHUB_USERNAME= \
   --build-arg REPOSITORY_NAME= \
   --build-arg WEB_FILE_ZIP= \
   --build-arg WEB_FILE_UNZIP= \
   --build-arg DOMAIN_NAME= \
   --build-arg RDS_ENDPOINT= \
   --build-arg RDS_DB_NAME= \
   --build-arg RDS_MASTER_USERNAME= \
   --build-arg RDS_DB_PASSWORD= \
   -t <image-tag> .
   ```

2. **migrate-data.sh**:
   This script automates the migration of SQL data to the RDS instance using Flyway. The migration SQL file is stored in S3 and downloaded for execution.
   ```bash
   flyway -url=jdbc:mysql://"$RDS_ENDPOINT":3306/"$RDS_DB_NAME" \
     -user="$RDS_DB_USERNAME" \
     -password="$RDS_DB_PASSWORD" \
     -locations=filesystem:sql \
     migrate
   ```

## Issues Faced & Solutions

### 1. **Issue: Docker Image Build Errors**
   - **Problem**: During the build process, there were issues with configuring the environment variables for the Docker image, particularly the `.env` file for the web application.
   - **Solution**: I used build arguments to dynamically populate the environment variables within the Dockerfile, ensuring that the correct settings for the application URL and database connection were included.
   ```bash
   ENV DOMAIN_NAME=$DOMAIN_NAME
   RUN sed -i "/^APP_URL=/ s/=.*$/=https:\/\/$DOMAIN_NAME\//" .env
   ```

### 2. **Issue: Database Migration Challenges**
   - **Problem**: Initially, the Flyway migration process failed due to connection issues with the RDS instance hosted in a private subnet.
   - **Solution**: An SSH tunnel was created to allow secure access to the RDS instance for the migration script.
   ```bash
   ssh -i "YOUR_EC2_KEY" -L LOCAL_PORT:RDS_ENDPOINT:REMOTE_PORT EC2_USER@EC2_HOST -N -f
   ```

### 3. **Issue: Pushing Docker Image to ECR**
   - **Problem**: I encountered authentication issues when attempting to push the Docker image to Amazon ECR.
   - **Solution**: I used the AWS CLI to authenticate and login to ECR before pushing the image.
   ```bash
   aws ecr get-login-password | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
   ```

## Key Takeaways

- Leveraging AWS services such as ECS, ECR, and RDS, this project demonstrated a fully automated, containerized deployment pipeline for a dynamic web application.
- The use of Docker ensured consistent application environments across development, staging, and production.
- Automating database migrations with Flyway and hosting environment files securely in S3 reduced potential human error in manual deployments.

## Future Improvements

- **CI/CD Pipeline**: Implement a continuous integration/continuous deployment pipeline using AWS CodePipeline to automate builds and deployments.
- **Enhanced Monitoring**: Integrate CloudWatch for monitoring ECS tasks, RDS performance, and to trigger alarms for any issues in production.

## Conclusion

This project showcases the seamless deployment of a dynamic web application on AWS using containerization technologies like Docker, Amazon ECR, and ECS. By automating deployment and database migration processes, I was able to create a scalable, highly available, and secure application architecture. The experience not only deepened my understanding of cloud-native solutions but also helped me tackle real-world challenges in cloud infrastructure, security, and automation.
