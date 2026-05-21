Bilal, aapka yeh step-by-step troubleshooting safar waqai lajawab hai! Ek aam tutorial sirf seedha rasta dikhata hai, lekin ek **acha DevOps engineer** hamesha "Problem -> Troubleshooting -> Solution" ka poora logic samajhta hai.

Maine aapke is poore journey ko ek behtareen, highly professional aur scannable **`README.md`** file mein badal diya hai. Isme aapke bataye huye saare points, region ko variables mein dalne ki behtar practice, aur real-world caching ka logic bohot khoobsurat andaz mein shamil kiya hai.

Aap niche diye gaye pure block ko copy karke apni `README.md` file mein daal sakte hain:

---

```markdown
# 🚀 Automated Static Website Deployment using AWS S3, CloudFront & GitHub Actions

An production-grade, serverless CI/CD pipeline that automatically deploys a static portfolio website to AWS S3, distributes it globally via CloudFront CDN, and instantly clears the edge caches using GitHub Actions.

This project highlights a real-world DevOps workflow, focusing on **Infrastructure integration**, **Cache optimization**, and **Automated resource synchronization**.

---

## 🛠️ Architecture & Workflow Overview
1. **Developer** pushes code to the `main` branch of a GitHub Repository.
2. **GitHub Actions** triggers a workflow that authenticates with AWS using IAM secrets.
3. **AWS S3** receives the latest web assets via automated synchronization (`aws s3 sync`).
4. **AWS CloudFront** serves the website globally with low latency.
5. **Cache Invalidation** automatically clears outdated edge locations so users instantly see the newest updates.

---

## 🚀 Step-by-Step Implementation Guide

### Step 1: Local Setup & Basic HTML
1. Create a dedicated folder on your local machine and open it in **VS Code**.
2. Create an `index.html` file with the following baseline code:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>My DevOps Portfolio</title>
</head>
<body>
    <h1>Welcome to my DevOps Portfolio</h1>
    <p>This site is deployed automatically using GitHub Actions.</p>
</body>
</html>

```

3. Test it locally by opening the file directly in any browser to ensure it displays correctly.

### Step 2: Initialize Git & Push to GitHub

Create a new GitHub repository (e.g., `my-portfolio`) and execute the following commands in your terminal to push your initial files:

```bash
git init
git add .
git commit -m "First commit: website files"
git branch -M main
git remote add origin <your-github-repo-url>
git push -u origin main

```

### Step 3: Provision AWS Infrastructure

Log into your AWS Management Console and configure the following services:

1. **Amazon S3**: Create a bucket to host your static website assets.
2. **Amazon CloudFront**: Create a global distribution pointing to your S3 bucket as the origin.
3. **AWS IAM**: Create a programmatic deployment user with the following managed policies attached:
* `AmazonS3FullAccess`
* `CloudFrontFullAccess`


4. Generate an **Access Key ID** and **Secret Access Key** for this IAM user.

### Step 4: Configure GitHub Secrets & Variables

To securely pass your AWS credentials and resource attributes to the automation pipeline without hardcoding them, navigate to **Settings -> Secrets and variables -> Actions** in your GitHub repository and add the following:

| Type | Name | Description / Example Value |
| --- | --- | --- |
| **Secret** | `AWS_ACCESS_KEY_ID` | Programmatic IAM access key |
| **Secret** | `AWS_SECRET_ACCESS_KEY` | Programmatic IAM secret key |
| **Secret** | `AWS_S3_BUCKET_NAME` | Name of your target S3 bucket |
| **Secret** | `AWS_CLOUDFRONT_DISTRIBUTION_ID` | Alphanumeric CloudFront ID (e.g., `EUM5MASNP7JN1`) |
| **Variable** | `AWS_REGION` | Target deployment region (e.g., `ap-south-1`) |

---

## 🧠 The DevOps Journey: Troubleshooting & Optimization Logs

During development, the infrastructure encountered two classic real-world bottlenecks. Here is how they were detected and engineered out:

### 🔍 Issue 1: CloudFront `Access Denied` Error

* **The Symptom**: When visiting the CloudFront domain name (`*.cloudfront.net`), the page threw an Access Denied error.
* **The Root Cause**: CloudFront did not know which root file to search for when hitting the top-level URL path.
* **The Fix**:
1. Open the CloudFront Console, select your distribution, and click **Edit**.
2. Locate the **Default root object** field and explicitly type: `index.html`.
3. Save changes. The domain will now correctly load the landing page.



### 🔍 Issue 2: Updates Stalled (Stale Edge Cache)

* **The Symptom**: After updating `index.html` text content and pushing it to GitHub, the pipeline ran successfully, but refreshing the CloudFront link still showed the old content.
* **The Root Cause**: CloudFront edge locations cache static files globally to maximize performance. Without an invalidation request, the old cache remains active until its Time-To-Live (TTL) expires.
* **The Fix**: Integrate an automated cache invalidation command directly inside the CI/CD pipeline using the unique distribution ID.

---

## 🤖 The Final Optimized CI/CD Pipeline

Create a file named `.github/workflows/deploy.yml` and paste the production code below.

> 💡 **Best Practice Note:** This workflow leverages GitHub Repository Variables for the `aws-region` to avoid brittle hardcoding, and includes a `paths-ignore` rule to prevent unnecessary build runs when simply tweaking documentation.

```yaml
name: Deploy Website to AWS S3 and CloudFront

on:
  push:
    branches:
      - main
    paths-ignore:
      - 'README.md'

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    # 1. Checkout repository code onto the GitHub runner environment
    - name: Checkout code
      uses: actions/checkout@v4

    # 2. Configure AWS CLI credentials dynamically using variable parameters
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ vars.AWS_REGION }}

    # 3. Synchronize project files directly to the S3 bucket excluding git metadata
    - name: Deploy to S3
      run: |
        aws s3 sync . s3://${{ secrets.AWS_S3_BUCKET_NAME }} --exclude ".git*" --exclude ".github*"

    # 4. Invalidate CloudFront edge caches to instantly deliver updates globally
    - name: Invalidate CloudFront Cache
      run: |
        aws cloudfront create-invalidation --distribution-id ${{ secrets.AWS_CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"

```

To update your deployment stack with this pipeline, push it via terminal:

```bash
git add .github/workflows/deploy.yml
git commit -m "ci: add automated CloudFront cache invalidation step"
git push origin main

```

Refreshed browsers will now dynamically pull the new message instantly:

> *"Welcome to my DevOps Portfolio. This website is hosted securely on AWS S3 + CloudFront and updated automatically via GitHub Actions CI/CD pipeline."*

---

## 🧼 Clean-up Protocol (Post-Lab Cost Optimization)

To avoid any accidental AWS billing charges after completing this laboratory exercise, completely dismantle the temporary sandbox resources in this exact order:

1. **CloudFront**: Select your distribution, click **Disable**, wait 2 minutes for deployment propagation, then select it again and click **Delete**.
2. **Amazon S3**: Empty your portfolio bucket completely, then proceed to **Delete** the bucket container.
3. **AWS IAM**: Delete the generated Access Keys, then remove the deployment user profile entirely.
4. **GitHub Secrets**: Clear out your stored repository deployment credentials.

```

***

### 🌟 Bilal Aapke Liye Pro-Tip:
Is README mein maine `vars.AWS_REGION` add kiya hai (jo ke dynamic environment variable hai). Aap bas GitHub repository mein ja kar jahan secrets banaye hain, wahan upar ek **Variables** ka tab hoga, usme `AWS_REGION` naam ka variable bana kar uski value `ap-south-1` set kar dein. Yeh aapke resume/portfolio par dekhne walo ko bohot professional lagega!

```
