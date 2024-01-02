
### Prerequisites:

1. **Amazon Web Services (AWS) Account:**
   - Ensure you have an AWS account. If not, you can sign up for one [here](https://aws.amazon.com/).

2. **AWS CLI (Command Line Interface):**
   - Install the AWS CLI on your local machine to interact with AWS services. You can download it from [here](https://aws.amazon.com/cli/).

3. **Docker:**
   - Install Docker on your local machine. You can download it from [here](https://www.docker.com/get-started).

### Steps:

1. **Create an EC2 Instance:**
   - Log in to the AWS Management Console.
   - Navigate to the EC2 service.
   - Click on "Launch Instance" and choose an Amazon Machine Image (AMI). For example, you can use Amazon Linux or Ubuntu.
   - Select an instance type and configure the instance details, including security groups and key pairs.

2. **Connect to the EC2 Instance:**
   - Use the AWS CLI or an SSH client to connect to your EC2 instance. For example:
     ```bash
     ssh -i your-key-pair.pem ec2-user@your-ec2-instance-ip
     ```

3. **Install Docker and Nginx on the EC2 Instance:**
   - Update the package index, install Docker, and install Nginx:
     ```bash
     sudo yum update -y
     sudo amazon-linux-extras install docker -y
     sudo service docker start
     sudo usermod -a -G docker ec2-user
     sudo amazon-linux-extras install nginx1 -y
     sudo service nginx start
     ```

4. **Create a Dockerfile:**
   - Create a `Dockerfile` in your project directory. Here's a simple example for a Node.js application:
     ```Dockerfile
     FROM node:14-alpine
     WORKDIR /app
     COPY package*.json ./
     RUN npm install
     COPY . .
     EXPOSE 80
     CMD ["npm", "start"]
     ```

5. **Build and Run Docker Container:**
   - Build the Docker image and run the container on your EC2 instance:
     ```bash
     docker build -t your-image-name .
     docker run -p 8080:80 -d your-image-name
     ```

6. **Configure Nginx as a Reverse Proxy:**
   - Open the Nginx configuration file for editing:
     ```bash
     sudo nano /etc/nginx/nginx.conf
     ```
   - Add the following `location` block inside the `server` block to configure Nginx as a reverse proxy:
     ```nginx
     server {
         # other server configurations...

         location / {
             proxy_pass http://localhost:8080; # Use the port where your Docker container is running
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection 'upgrade';
             proxy_set_header Host $host;
             proxy_cache_bypass $http_upgrade;
         }

         # other location configurations...
     }
     ```
   - Save the file and restart Nginx:
     ```bash
     sudo service nginx restart
     ```

7. **Access the Single Page Application:**
   - Open your web browser and navigate to your EC2 instance's public IP address. You should see your single-page application running through Nginx.

Remember to replace placeholders like `your-key-pair.pem`, `your-ec2-instance-ip`, and `your-image-name` with your actual values. Adjust the Nginx configuration based on your specific requirements.
