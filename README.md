<!-- @format -->
# AWS Secure Static Website Hosting 🚀

This project demonstrates hosting a secure static website on AWS using:

Amazon S3 (Private) + CloudFront (OAI) + WAF + CloudWatch Logs + SNS + CloudWatch Alarm + Cost Explorer & Budgets
## 📂 Architecture Diagram
<div align="center">
      <img src="Project01/Images/Architechture of this project.png" width=100%>
</div>
  
The website is fully private, served securely through CloudFront protected with AWS WAF, monitored via CloudWatch and cost-managed with Budgets and Cost Explorer.
## 📋 Detailed Step-by-Step Setup

### Step 1: Create a Private S3 Bucket
1.	Navigate to AWS Console → S3 → Create bucket
2.	Bucket name: awsfirst-project
3.	Block Public Access: Enable all options ✅
4.	Create bucket
5.  Upload Website Files:
•	Upload index.html, CSS, JS, images
•	Files remain private (no public access)
### Step 2: Create CloudFront Distribution with OAI
1.	Go to CloudFront → Create distribution
2.	Origin domain: Choose your S3 bucket
3.	Origin access:
•	Select Origin Access Identity (OAI)
•	Create new OAI →
         Update bucket policy automatically ✅
5.	Viewer protocol policy:
         Redirect HTTP to HTTPS
7.	Default root object: index.html
8.	Create distribution and wait until Deployed

### Test:
•	Visit your CloudFront domain 
  ```url
   (e.g., https://dXXXX.cloudfront.net)
  ```
•	You should see your index.html

### Step 3: Attach AWS WAF to CloudFront
1.	Go to WAF & Shield → Web ACLs → Create web ACL
2.	Region: Global (CloudFront)
3.	Resources to protect: Select your CloudFront distribution
4.	Add AWS Managed Rules:
          •	AWSManagedRulesCommonRuleSet (common attacks)
          •	AWSManagedRulesKnownBadInputsRuleSet (SQL Injection, XSS)
5.	Create Web ACL
✅ Your website is now protected from SQL Injection and XSS attacks.

### Step 4: Enable CloudWatch Logs for WAF
1.	Navigate to WAF → Your Web ACL → Logging and metrics tab
2.	Click Enable logging → CloudWatch Logs
3.	Create log group: aws-waf-logs-webacl-first
4.	Enable Store full logs ✅

Sample Log JSON:
{
  "action": "BLOCK",
  "httpSourceIp": "103.15.120.50",
  "httpUri": "/admin.php",
  "country": "BD",
  "terminatingRuleId": "AWS-AWSManagedRulesCommonRuleSet"
}

### Step 5: Set Up Amazon SNS for Notifications (Independent)
1.	Go to Amazon SNS → Create topic
            •	Type: Standard
            •	Name: WAFNotifications
2.	Create subscription →
             Protocol:  Email → Add your email
4.	Check your inbox and Confirm subscription
5.	You can now manually publish notifications or link with alarms later if needed

✅ SNS is now ready to send notifications separately.

### Step 6: Create a CloudWatch Alarm for WAF Activity
1.	Go to CloudWatch → Alarms → Create alarm
2.	Select metric:
      •	WAFV2 → WebACL → BlockedRequests
3.	Threshold example:
      •	Trigger if >= 60 requests within 1 minutes
4.	Alarm action:
      • Send notification to your existing SNS topic (WAFNotifications)
5.	Alarm name: ```WAF-BlockedRequests-High``` → Create alarm

✅ You will receive email alerts whenever the blocked requests exceed the threshold.

### Step 7: Enable Cost Explorer and AWS Budgets
1.	Billing → Cost Explorer → Enable Cost Explorer
2.	Budgets → Create budget → Cost budget
   •	Example: $5 per month
o	Add your email for budget alerts
3.	Monitor usage and spending in Cost Explorer dashboard
✅ You will receive alerts if AWS spending exceeds your defined budget.

### Project Output
•	Secure Static Website URL via CloudFront (HTTPS)
•	WAF Protection with blocked requests (403 Forbidden)
•	CloudWatch Logs with request details
•	SNS Notifications for alerts
•	CloudWatch Alarm for blocked traffic spikes
•	Cost Management with AWS Budgets and Cost Explorer

### Problem Statement ⚠️

During the implementation of this AWS Secure Static Website Hosting project, two main challenges were encountered:

1️⃣**CloudFront did not serve the ``index.html`` file**

•	**Issue:** After creating the CloudFront distribution, the website URL showed Access Denied or a blank page instead of loading ``index.html``.
•	**Cause:** CloudFront requires a Default Root Object to be set ``(e.g., index.html)`` when serving content from a private S3 bucket via OAI.

**Solution:**
•	Updated the CloudFront Distribution → Default Root Object to ``index.html``
•	After deployment, the website loaded successfully.

2️⃣**CloudWatch Logs did not capture WAF logs**
•	Issue: After enabling WAF logging, no logs appeared in CloudWatch initially.
•	Cause: WAF logging requires a specific log group naming format in CloudWatch for the connection to work properly.

**Solution:**
•	Created a CloudWatch log group with the format:
•	```aws-waf-logs-<name>```
**Example:** ```aws-waf-logs-securewebsite```
•	Re-attached the log group to the WAF Web ACL and enabled full logging.
•	Logs started appearing in CloudWatch Logs as expected.

**Key Takeaways**
•	Always configure the Default Root Object in CloudFront to serve static websites correctly.
•	Use the correct log group naming format (aws-waf-logs-<name>) for WAF to successfully push logs to CloudWatch.

**This Problem Statement will help others avoid the same configuration mistakes when deploying a secure AWS static website.**
