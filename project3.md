# Deploy a Web Application Using Lambda, API Gateway, and DynamoDB

## Step 1: Set Up DynamoDB for Storage

1. **Create a DynamoDB Table**:
   - Navigate to the **DynamoDB Console**.
   - Click **Create table**.
   - Provide a table name (e.g., `WebAppTable`).
   - Set a **Primary Key** (e.g., `id` of type `String`).
   - Leave other settings as default for now (On-Demand capacity mode is preferred to manage traffic automatically).
   - Click **Create** to create the table.

2. **Add Items to DynamoDB** (Optional):
   - In the **Items** tab, you can manually add some entries for testing purposes, each with a unique `id` and other attributes relevant to your application (e.g., `name`, `age`, `status`, etc.).

---

## Step 2: Create a Lambda Function for Business Logic

1. **Create a Lambda Function**:
   - Go to the **Lambda Console**.
   - Click **Create function**.
   - Choose **Author from scratch**.
   - Give the function a name (e.g., `WebAppLambda`).
   - Set the **Runtime** to `Node.js`, `Python`, or any language supported by AWS Lambda that you're familiar with.
   - Click **Create function**.

2. **Write the Lambda Code**:
   - In the **Code** section, write your Lambda function to interact with DynamoDB.
   - For example, in **demo.py**, a simple function to retrieve items from DynamoDB could look like this:

  ```python
import json
import os
import boto3

def lambda_handler(event, context):
    try:
        mypage = page_router(event['httpMethod'], event['queryStringParameters'], event['body'])
        return mypage
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }

def page_router(httpmethod, querystring, formbody):
    if httpmethod == 'GET':
        try:
            with open('contactus.html', 'r') as htmlFile:
                htmlContent = htmlFile.read()
            return {
                'statusCode': 200,
                'headers': {"Content-Type": "text/html"},
                'body': htmlContent
            }
        except Exception as e:
            return {
                'statusCode': 500,
                'body': json.dumps({'error': str(e)})
            }

    elif httpmethod == 'POST':
        try:
            insert_record(formbody)
            with open('success.html', 'r') as htmlFile:
                htmlContent = htmlFile.read()
            return {
                'statusCode': 200,
                'headers': {"Content-Type": "text/html"},
                'body': htmlContent
            }
        except Exception as e:
            return {
                'statusCode': 500,
                'body': json.dumps({'error': str(e)})
            }

def insert_record(formbody):
    formbody = formbody.replace("=", "' : '")
    formbody = formbody.replace("&", "', '")
    formbody = "INSERT INTO WebAppTable value {'" + formbody + "'}"

    client = boto3.client('dynamodb')
    response = client.execute_statement(Statement=formbody)
    # Assuming the execute_statement call returns successfully
    return response

```

3. **Configure Lambda Permissions**:
   - Go to the **Permissions** tab and make sure the Lambda execution role has the necessary permissions to interact with DynamoDB.
   - Attach the **`AmazonDynamoDBFullAccess`** or a custom policy to allow DynamoDB interactions.
   
---

## Step 3: Set Up API Gateway for REST API

1. **Create an API in API Gateway**:
   - Navigate to the **API Gateway Console**.
   - Click **Create API** and choose **HTTP API** (if you want a simpler configuration) or **REST API** (for more features).
   - Click **Build**.

2. **Configure API Resources**:
   - Create a new **resource** (e.g., `/items`) to represent your API endpoint.
   - For the `/items` resource, create an **HTTP method** (e.g., `GET`) to fetch data.
   
3. **Integrate API Gateway with Lambda**:
   - In the **Integration** tab, choose **Lambda function** as the integration type.
   - Select the Lambda function you created earlier (`WebAppLambda`).
   - Grant API Gateway permissions to invoke the Lambda function.

4. **Enable CORS** (Optional):
   - In the **Settings** of your API method, enable **CORS** if your web application will be making cross-origin requests to the API.

5. **Deploy the API**:
   - Once you've configured the API, click **Deploy API**.
   - Create a new deployment stage (e.g., `prod` or `dev`).
   - After deployment, you will get an **Invoke URL** (something like `https://xyz.execute-api.region.amazonaws.com/prod/items`).

---

## Step 4: Create a Frontend for the Web Application(Inside Lambda Code Source)

1. **Set Up HTML/JavaScript Frontend**:
   - Create a basic HTML page (`index.html`) that interacts with the API using JavaScript (AJAX or Fetch API).
   
   Example:

  ```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Contact Form</title>
  <style>
    body {
      font-family: 'Arial', sans-serif;
      background-color: #f4f4f4;
      margin: 0;
      padding: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      height: 100vh;
    }

    form {
      background-color: #fff;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
      transition: transform 0.3s ease-in-out;
    }

    h2 {
      text-align: center;
      color: #333;
      margin-bottom: 20px;
    }

    label {
      display: block;
      margin: 10px 0 5px;
      color: #555;
    }

    input,
    textarea {
      width: 100%;
      padding: 10px;
      margin-bottom: 10px;
      box-sizing: border-box;
      border: 1px solid #ccc;
      border-radius: 4px;
    }

    input[type="submit"] {
      background-color: #4caf50;
      color: #fff;
      cursor: pointer;
      font-size: 16px; /* Increased text size for submit button */
    }

    input[type="submit"]:hover {
      background-color: #45a049;
    }

    /* Additional styles for animation */
    form:hover {
      transform: scale(1.05);
    }

    /* Additional styles for footer */
    footer {
      margin-top: 20px;
      font-size: 18px;
      text-align: center;
    }
  </style>
</head>
<body>

  <h2>Welcome to my AWS Project Task</h2>

  <form action="/dev" method="post">
    <h2>Contact Us</h2>
    <label for="fname">First Name:</label>
    <input type="text" id="fname" name="fname" required>

    <label for="lname">Last Name:</label>
    <input type="text" id="lname" name="lname" required>

    <label for="email">Email ID:</label>
    <input type="text" id="email" name="email" required>

    <label for="message">Message:</label>
    <textarea id="message" name="message" rows="4" cols="50" required></textarea>

    <input type="submit" value="Submit">
  </form>

  <!-- Simple JavaScript for animation -->
  <script>
    const form = document.querySelector('form');

    form.addEventListener('mouseover', () => {
      form.style.transform = 'scale(1.05)';
    });

    form.addEventListener('mouseout', () => {
      form.style.transform = 'scale(1)';
    });
  </script>

</body>
</html>
```
# Thank You Page HTML

This HTML code creates a responsive thank you page for your AWS project, featuring an animated message to the user.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Thank You</title>
  <style>
    body {
      font-family: 'Arial', sans-serif;
      background-color: #f4f4f4;
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }

    h2 {
      text-align: center;
      color: #333;
      padding: 20px;
      background-color: #fff;
      border-radius: 8px;
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
      opacity: 0;
      transform: translateY(20px);
      transition: opacity 0.5s ease-in-out, transform 0.5s ease-in-out;
    }

    /* Additional styles for animation */
    .visible {
      opacity: 1;
      transform: translateY(0);
    }
  </style>
</head>
<body>

  <h2 id="thankYouMessage">Thanks for trying this Project. You can verify data in the DynamoDB Table.</h2>

  <!-- Simple JavaScript for animation -->
  <script>
    const thankYouMessage = document.getElementById('thankYouMessage');

    // Adding a delay before showing the message
    setTimeout(() => {
      thankYouMessage.classList.add('visible');
    }, 500);
  </script>

</body>
</html>
