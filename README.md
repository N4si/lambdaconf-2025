# 🧠 AI-Powered Receipt Processor with AWS Lambda & Textract

This project demonstrates how to build a **serverless AI receipt processing pipeline** using:

- **AWS Lambda** for orchestration
- **Amazon Textract** for extracting structured data from receipts
- **Amazon DynamoDB** for storing parsed data
- **Amazon S3** for uploading receipt images
- **Amazon SES** for email notifications

---

## 📦 Repository

GitHub Code: [https://github.com/N4si/lambdaconf-2025](https://github.com/N4si/lambdaconf-2025)  
Language: **Python 3.8+**  
Deployment: All steps done through **AWS Console**

---

## ✅ Prerequisites

Before setting up, ensure the following:

- AWS account with billing enabled
- AWS services access:
  - Lambda, S3, Textract, DynamoDB, SES, IAM, CloudWatch
- AWS Console access with admin permissions
- Verified **sender and recipient emails** in SES (sandbox restriction)
- No need to zip or deploy code — everything is in `lambda_function.py`

---

## 🛠️ Setup: Step-by-Step (Using AWS Console)

### 🔸 Step 1: Create DynamoDB Table

1. Navigate to **DynamoDB > Tables > Create Table**
2. Set table name: `Receipts`
3. Primary key: `receipt_id` (type: String)
4. Leave other settings as default and click **Create Table**

---

### 🔸 Step 2: Verify Emails in Amazon SES

1. Go to **Amazon SES > Email Addresses**
2. Click **Verify a New Email Address**
3. Verify:
   - Sender email (used by Lambda to send emails)
   - Recipient email (where the receipt summary is sent)
4. Confirm the verification by clicking the link sent to your inbox

> ⚠️ SES is in sandbox by default. You can only send to verified email addresses.

---

### 🔸 Step 3: Create an S3 Bucket

1. Go to **Amazon S3 > Create Bucket**
2. Name it something like: `receipt-upload-demo`
3. Keep default settings (block public access enabled)
4. Click **Create Bucket**

---

### 🔸 Step 4: Create IAM Role for Lambda

1. Navigate to **IAM > Roles > Create Role**
2. Choose **Lambda** as the trusted entity
3. Attach a custom inline policy like this:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": ["s3:*"], "Resource": "*" },
    { "Effect": "Allow", "Action": ["textract:AnalyzeExpense"], "Resource": "*" },
    { "Effect": "Allow", "Action": ["dynamodb:PutItem"], "Resource": "arn:aws:dynamodb:*:*:table/Receipts" },
    { "Effect": "Allow", "Action": ["ses:SendEmail"], "Resource": "*" },
    { "Effect": "Allow", "Action": ["logs:*"], "Resource": "*" }
  ]
}
````

4. Name the role: `LambdaReceiptRole` and create it.

---

### 🔸 Step 5: Create the Lambda Function

1. Go to **Lambda > Create Function**
2. Select **Author from Scratch**
3. Name: `receiptProcessor`
4. Runtime: **Python 3.8 or 3.9**
5. Choose the role created in Step 4 (`LambdaReceiptRole`)
6. Click **Create Function**

---

### 🔸 Step 6: Add the Code

1. In your new Lambda function, scroll to **Code source**
2. Click **Edit** and paste the contents of `lambda_function.py` from this repo
3. Click **Deploy**

---

### 🔸 Step 7: Set Environment Variables

Navigate to **Configuration > Environment Variables > Edit**
Add the following:

| Key                   | Value                                 |
| --------------------- | ------------------------------------- |
| `DYNAMODB_TABLE`      | `Receipts`                            |
| `SES_SENDER_EMAIL`    | `your_verified_sender@example.com`    |
| `SES_RECIPIENT_EMAIL` | `your_verified_recipient@example.com` |

Click **Save**.

---

### 🔸 Step 8: Add S3 Trigger to Lambda

1. Go to **Configuration > Triggers > Add Trigger**
2. Select **S3**
3. Choose your bucket (`receipt-upload-demo`)
4. Event type: **All object create events**
5. Click **Add**

---

## 📈 How It Works

1. User uploads a receipt image to S3 (`.jpg`, `.png`)
2. S3 triggers the Lambda function
3. Lambda:

   * Calls **Textract AnalyzeExpense** to extract:

     * Vendor name
     * Date
     * Total
     * Line items (name, quantity, price)
   * Stores structured data in **DynamoDB**
   * Sends formatted HTML email via **SES**

---

## 📬 Sample Email Output

* Subject: `Receipt Processed: Walmart - $42.18`
* Body includes:

  * Receipt ID
  * Vendor, Date, Total
  * List of items
  * S3 path to the image

---

## 🪵 Debugging with CloudWatch Logs

To troubleshoot:

1. Go to **CloudWatch > Log Groups**
2. Find: `/aws/lambda/receiptProcessor`
3. Look for logs:

   * `"Processing receipt from ..."`
   * `"Textract analyze_expense call successful"`
   * `"Receipt data stored in DynamoDB"`
   * `"Email notification sent"`

---

## 💡 Notes

* Works best with clean, legible receipts
* Textract doesn't require templates — it uses machine learning
* You can improve this project by:

  * Adding a web interface for uploads
  * Classifying receipts into categories
  * Tagging expenses by project or department

---

## 🔗 Related AWS Docs

* [Amazon Textract AnalyzeExpense](https://docs.aws.amazon.com/textract/latest/dg/analyzing-expense-documents.html)
* [Lambda with S3 Triggers](https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html)
* [DynamoDB Basics](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GettingStarted.html)
* [Amazon SES Email Sending](https://docs.aws.amazon.com/ses/latest/dg/send-email.html)

---

## 🙌 Credits

Developed for **LambdaConf 2025**
By [@N4si](https://github.com/N4si)
📧 Questions? Open an issue or drop a message on LinkedIn!
