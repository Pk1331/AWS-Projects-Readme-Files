**AWS CloudFront** with an **Application Load Balancer (ALB)** to deliver a simple HTML page using a **Linux instance**
# AWS CloudFront with Application Load Balancer (ALB) to Serve Simple HTML from Linux Instance

## Step 1: Set Up a Linux EC2 Instance

1. **Launch an EC2 Instance**:
   - Go to the EC2 dashboard in AWS.
   - Launch a new **Linux EC2 instance** (e.g., Amazon Linux 2 or Ubuntu).
   - Ensure the instance is in a **public subnet** with a **security group** allowing HTTP (port 80) traffic.

2. **Connect to Your EC2 Instance**:
   - Use SSH to connect to your EC2 instance:
     ```bash
     ssh -i your-key.pem ec2-user@your-ec2-public-ip
     ```

3. **Install a Web Server (Apache)**:
   - Install Apache on the EC2 instance:
     ```bash
     sudo yum update -y  # For Amazon Linux
     sudo yum install httpd -y  # For Amazon Linux
     ```

4. **Create a Simple HTML Page**:
   - Create an HTML page that will be served by Apache:
     ```bash
     echo "<html><body><h1>Hello from EC2</h1></body></html>" | sudo tee /var/www/html/index.html
     ```

5. **Start and Enable Apache**:
   - Start and enable the Apache service:
     ```bash
     sudo systemctl start httpd
     sudo systemctl enable httpd
     ```

6. **Test the Setup**:
   - In your browser, navigate to your EC2 instance's public IP (`http://<your-ec2-public-ip>`) and ensure you see the message **Hello from EC2**.

---

## Step 2: Set Up the Application Load Balancer (ALB)

1. **Create an ALB**:
   - Go to the **EC2 Dashboard**, click **Load Balancers**, then **Create Load Balancer**.
   - Choose **Application Load Balancer**.
   - Set the following:
     - **Name**: Choose a name for the ALB.
     - **Scheme**: Internet-facing.
     - **Listeners**: HTTP (Port 80).
     - **VPC**: Choose the same VPC as your EC2 instance.
     - **Subnets**: Choose two public subnets for the ALB.

2. **Create a Target Group**:
   - Go to the **Target Groups** section and click **Create Target Group**.
   - Choose **Instances** as the target type.
   - Set the following configurations:
     - **Name**: Give your target group a name.
     - **Protocol**: HTTP.
     - **Port**: 80.
     - **VPC**: Choose the same VPC as your EC2 instance.
     - **Health Check Path**: `/index.html`.
   - Click **Next** and register your EC2 instance to this target group.

3. **Add Target Group to Load Balancer**:
   - After creating the target group, return to the ALB creation flow.
   - In the **Listener and Routing** section, select the previously created **Target Group**.

4. **Create the Load Balancer**:
   - Review and create the ALB.

5. **Test the ALB**:
   - Once the ALB is created, check its **DNS Name** (found in the ALB details).
   - Access the ALB by navigating to its DNS name (`http://<alb-dns-name>`) in your browser. You should see the **Hello from EC2** message.

---

## Step 3: Set Up CloudFront Distribution

1. **Create a CloudFront Distribution**:
   - Navigate to **CloudFront** in the AWS Console.
   - Click **Create Distribution** and select **Web**.

2. **Configure Origin Settings**:
   - **Origin Domain Name**: Enter your **ALB DNS name** (from the previous step).
   - **Origin Protocol Policy**: Choose **HTTP only** (since ALB serves over HTTP).

3. **Default Cache Behavior**:
   - **Viewer Protocol Policy**: Choose **Redirect HTTP to HTTPS** (recommended for security).
   - **Allowed HTTP Methods**: Select **GET, HEAD** (for simple HTML delivery).

4. **Create the Distribution**:
   - Review the settings and create the CloudFront distribution.
   - Wait for the status to become **Deployed** (this may take a few minutes).

---

## Step 4: Test CloudFront Setup

1. **Access Your CloudFront Distribution**:
   - Once the CloudFront distribution is deployed, navigate to the **CloudFront domain name** (e.g., `https://<cloudfront-domain-name>`) in your browser.
   - You should see the **Hello from EC2** message served through CloudFront, ALB, and your EC2 instance.

---

## Summary

This process configures:
- A **Linux EC2 instance** hosting a simple HTML page.
- An **Application Load Balancer (ALB)** distributing traffic across the EC2 instance.
- **CloudFront**, which caches the content for faster global delivery and improved scalability.
```
