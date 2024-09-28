# Deliver a Static Website Using S3 (With Blocked Public Access), CloudFront, Route 53, and ACM

## Step 1: Set Up an S3 Bucket for Static Website Hosting (with Public Access Blocked)

1. **Create an S3 Bucket**:
   - Go to the **S3 Console**.
   - Click **Create bucket** and provide a globally unique name (e.g., `example.com`).
   - Choose the region.
   - **Enable Block All Public Access**. This ensures that no one can directly access your S3 bucket.
   - Click **Create bucket**.

2. **Upload Your Website Files**:
   - Upload your HTML, CSS, JS, and any other assets to the S3 bucket.

---

## Step 2: Set Up CloudFront with Origin Access Control (OAC)

1. **Create CloudFront Distribution**:
   - Navigate to the **CloudFront Console** and click **Create Distribution**.
   - Choose **Web**.

2. **Configure Origin Settings**:
   - **Origin Domain Name**: Enter your **S3 bucket URL** (it should look like `example.com.s3.amazonaws.com`).
   - **Origin Access**: Choose **Origin Access Control (OAC)**.
     - Click **Create Control Setting** if it is your first time.
     - Configure OAC by giving it a name and leaving **Enable signed requests** checked.
   - **Origin Protocol Policy**: Set it to **HTTPS only**.

3. **Update S3 Bucket Permissions**:
   - Go to your **S3 bucket** in the AWS Console.
   - Go to **Permissions** > **Bucket Policy**, and add the following policy to allow CloudFront access through OAC:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Principal": {
             "Service": "cloudfront.amazonaws.com"
           },
           "Action": "s3:GetObject",
           "Resource": "arn:aws:s3:::example.com/*",
           "Condition": {
             "StringEquals": {
               "AWS:SourceArn": "arn:aws:cloudfront::YOUR_ACCOUNT_ID:distribution/YOUR_DISTRIBUTION_ID"
             }
           }
         }
       ]
     }
     ```
   - Replace `example.com` with your S3 bucket name, `YOUR_ACCOUNT_ID` with your AWS account ID, and `YOUR_DISTRIBUTION_ID` with your CloudFront distribution ID.
   
4. **Set Cache Behavior**:
   - **Viewer Protocol Policy**: Set to **Redirect HTTP to HTTPS** (for security).
   - **Allowed HTTP Methods**: Choose **GET, HEAD** (since it's a static website).

5. **SSL/TLS Settings**:
   - Under **SSL Certificate**, choose **Custom SSL Certificate**.
   - You’ll select an ACM certificate here (created in the next step).

6. **Create the Distribution**:
   - Click **Create Distribution** and wait for the status to become **Deployed**.

---

## Step 3: Obtain an SSL Certificate Using AWS Certificate Manager (ACM)

1. **Request a Public SSL Certificate**:
   - Go to **AWS Certificate Manager (ACM)**.
   - Click **Request a certificate**.
   - Enter your domain name (e.g., `example.com` and `www.example.com`).
   - Choose **DNS validation**.
   - Request the certificate.

2. **Validate the Certificate**:
   - If you're using **Route 53**, you can easily validate the domain by selecting **Create DNS record in Route 53** from ACM.
   - Once DNS records are created and validated, your certificate will be issued.

---

## Step 4: Set Up Route 53 for Domain Management

1. **Create a Hosted Zone in Route 53**:
   - Go to **Route 53** > **Hosted Zones** and click **Create hosted zone**.
   - Enter your domain name (e.g., `example.com`) and choose **Public hosted zone**.

2. **Create an A Record for CloudFront**:
   - Inside your hosted zone, click **Create record**.
   - Choose **Simple routing** and click **Next**.
   - Create an **A record** (or **Alias record**) that points to your CloudFront distribution.
   - Select **Alias to CloudFront distribution** and choose your CloudFront distribution from the dropdown.

---

## Step 5: Test the Setup

1. **Access Your Website**:
   - Once everything is set up, navigate to your domain (e.g., `https://example.com`).
   - Your website should be served through CloudFront, using the S3 bucket as the origin.

---

## Summary

- **S3 bucket** stores the static website with public access blocked.
- **CloudFront** serves content using **Origin Access Control (OAC)** to securely retrieve objects from the S3 bucket.
- **Route 53** manages the domain name, linking it to CloudFront.
- **ACM** provides the SSL certificate to enable HTTPS for your domain.
