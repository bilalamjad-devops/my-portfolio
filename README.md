---

What we want to achieve:
When developer commits changes, GitHub Actions should be run. It deployes the new code in S3 static website hosting. Cloudfront has to serve the data instantly.
Enhancement: We can add Route53 for domain and ACM for https. 
What we will do:
First we run our code locally to see. Then we push our code to GitHub. Then we create S3, Cloudfront, IAM User and attach policyes with him. Then write GitHub Actions file. 
Steps:
Local Development
Push to GitHub repository
Create S3, Cloudfront and IAM user
Store secrets in GitHub repository
GitHub Actions file and Testing
Commit

Local Development

Make a folder and open in VS code
Create index.html and paste code
Click index browser icon and open it

<!DOCTYPE html>
<html>
<head>
    <title>My DevOps Portfolio</title>
</head>
<body>
    <h1>Welcome to my DevOps Portfolio</h1>
    <p>This site is deployed automatically using GitHub Actions.</p>
</body>
</html>

---

2. Push to GitHub repository
Create new repository
git init, add, commit, branch, remote add origin, push

git init
git add .
git commit -m "First commit: website files"
git branch -M main
git remote add origin <your-github-repo-url>
git push -u origin main

---

3. Create S3, Cloudfront and IAM user
S3:
Create bucket
Bucket name: my-devops-portfolio-xyz 
Keep rest options as it is
Create

Cloudfront:
Create a cloudfront distribution
Distribution name: my-portfolio-cdn
S3 origin: Your_bucket
Use monitoring mode
Keep rest options as it is
Create

User:
User name: github-actions-user
Attach policies: AmazonS3FullAccess, CloudFrontFullAccess
Securits credentials > Create access key > Command line interface
Keep rest options as it is
Please save credentials file
Create  

---

4. Store secrets in GitHub repository
Github repository > Settings
Secrets and variables > Actions > New repository secret
AWS_ACCESS_KEY_ID 
AWS_SECRET_ACCESS_KEY
AWS_S3_BUCKET_NAME

---

5. GitHub Actions file and Testing 
Make .github/workflows/deploy.yml
Paste the file content
git add, commit and push
Paste your cloudfront distribution domain name

name: Deploy Website to AWS S3 and CloudFront

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    # 1. Checkout (download) your repository code onto the GitHub runner
    - name: Checkout code
      uses: actions/checkout@v4

    # 2. Configure the AWS CLI using your stored GitHub repository secrets
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ap-south-1 # Change this if your S3 bucket is in a different region

    # 3. Automatically sync and upload HTML files from GitHub to the target S3 bucket
    - name: Deploy to S3
      run: |
        aws s3 sync . s3://${{ secrets.AWS_S3_BUCKET_NAME }} --exclude ".git*" --exclude ".github*"
Please change your aws-region according to you. I should have created secret/variable also for it.
git add .github/workflows/deploy.yml
git commit -m "Added deploy.yml content"
git push origin main
Troubleshooting:
Question: What is the issue? 
Answer: We did not add index.html name which file our cloudfront from has to pull data.
Edit
Default root object: index.html
Save

---

6. Commit
Change content of website
git add, commit, push 
Paste/Refresh your cloudfront distribution domain name

Welcome to my DevOps Portfolio
his site is deployed automatically using GitHub Actions.
git add index.html
git commit -m "docs: update website content text for security and pipeline info"
git push origin main
Troubleshooting:
Question: What is the issue?
Answer: Cloudfront stores data in different edge locations to transfer data instantly. So, we need to remove cache. For removing cache, we have added new line of code to remove cache.
Github repository > Settings
Secrets and variables > Actions > New repository secret
AWS_CLOUDFRONT_DISTRIBUTION_ID: Your Cloudfront distribution ID
Update/Edit your deploy.yml file 
git add, commit, push
Paste/Refresh your cloudfront distrubtion domain name

name: Deploy Website to AWS S3 and CloudFront

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    # Configure AWS credentials and authenticate with the specified region
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ap-south-1

    # Sync project files directly to the target S3 bucket while excluding repository metadata
    - name: Deploy to S3
      run: |
        aws s3 sync . s3://${{ secrets.AWS_S3_BUCKET_NAME }} --exclude ".git*" --exclude ".github*"

    # Clear the CloudFront edge cache to instantly make updates visible to users
    - name: Invalidate CloudFront Cache
      run: |
        aws cloudfront create-invalidation --distribution-id ${{ secrets.AWS_CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
Please paste your aws-region according to you. 
git add .github/workflows/deploy.yml
git commit -m "ci: add automated CloudFront cache invalidation step"
git push origin main
After lab, please remove resources to avoid cost. 

H
A
H
