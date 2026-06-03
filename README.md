# 🚀 Static Website — AWS S3 + CloudFront + GitHub Actions CI/CD

A static website hosted on AWS S3 and served via CloudFront CDN, with automated deployments using GitHub Actions.

---

## 🏗️ Architecture

```
GitHub Actions → IAM User → S3 Bucket → CloudFront CDN → Users
```

| Service | Purpose |
|---|---|
| **AWS S3** | Static file storage & website hosting |
| **AWS CloudFront** | HTTPS CDN, caching & distribution |
| **AWS IAM** | Restricted CI/CD user permissions |
| **GitHub Actions** | Automated deploy on every push to `main` |

---

## 📁 Project Structure

```
your-project/
├── .github/
│   └── workflows/
│       └── deploy.yml      # GitHub Actions CI/CD workflow
├── index.html              # Main HTML file
├── style.css               # Styles
├── script.js               # JavaScript
├── assets/                 # Images, fonts, etc.
└── README.md               # You are here
```

---

## ⚙️ AWS Setup

### 1. S3 Bucket
- Static website hosting enabled
- `index.html` set as index document
- Public read access via bucket policy

### 2. CloudFront Distribution
- Origin: S3 website endpoint
- Viewer protocol: Redirect HTTP → HTTPS
- Default root object: `index.html`
- Cache invalidation on every deploy

### 3. IAM User (`github-actions-cicd`)
- Access restricted to **specific S3 bucket only**
- Access restricted to **specific CloudFront distribution only**
- No access to any other AWS services

#### IAM Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3ListAllBuckets",
      "Effect": "Allow",
      "Action": ["s3:ListAllMyBuckets"],
      "Resource": "*"
    },
    {
      "Sid": "S3BucketLevelAccess",
      "Effect": "Allow",
      "Action": ["s3:ListBucket", "s3:GetBucketLocation"],
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME"
    },
    {
      "Sid": "S3ObjectLevelAccess",
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    },
    {
      "Sid": "CloudFrontAccess",
      "Effect": "Allow",
      "Action": [
        "cloudfront:ListDistributions",
        "cloudfront:GetDistribution",
        "cloudfront:GetDistributionConfig",
        "cloudfront:CreateInvalidation",
        "cloudfront:GetInvalidation",
        "cloudfront:ListInvalidations"
      ],
      "Resource": [
        "*",
        "arn:aws:cloudfront::YOUR-ACCOUNT-ID:distribution/YOUR-DISTRIBUTION-ID"
      ]
    }
  ]
}
```

---

## 🔐 GitHub Secrets

Go to **GitHub Repo → Settings → Secrets and variables → Actions** and add:

| Secret Name | Description |
|---|---|
| `AWS_ACCESS_KEY_ID` | IAM user access key ID |
| `AWS_SECRET_ACCESS_KEY` | IAM user secret access key |
| `AWS_REGION` | AWS region (e.g. `ap-south-1`) |
| `S3_BUCKET_NAME` | Your S3 bucket name |
| `CLOUDFRONT_DISTRIBUTION_ID` | Your CloudFront distribution ID |

---

## 🔄 GitHub Actions Workflow

```yaml
name: Deploy Static Website to S3 + CloudFront

on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: Deploy to S3 & CloudFront
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Deploy to S3
        run: |
          aws s3 sync . s3://${{ secrets.S3_BUCKET_NAME }} \
            --delete \
            --exclude ".git/*" \
            --exclude ".github/*"

      - name: Invalidate CloudFront Cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

---

## 🚀 How to Deploy

Every push to the `main` branch automatically triggers a deployment.

```bash
# Make your changes
git add .
git commit -m "your commit message"
git push origin main
```

Then watch it live at **GitHub → Actions tab** ✅

---

## 🌍 Live URL

```
https://YOUR-CLOUDFRONT-DOMAIN.cloudfront.net
```

---

## 📋 Deployment Flow

```
Push to main branch
        ↓
GitHub Actions triggered
        ↓
AWS credentials configured
        ↓
Files synced to S3
        ↓
CloudFront cache invalidated
        ↓
🌍 Live on CloudFront URL
```

---

## 🛠️ Local Development

No build step required — just open `index.html` in your browser:

```bash
# Option 1: Open directly
open index.html

# Option 2: Use a local server
npx serve .
# or
python3 -m http.server 3000
```

---

## 📝 Notes

- Changes go live within **1–2 minutes** after pushing to `main`
- CloudFront cache is **automatically cleared** on every deploy
- The IAM user has **minimal permissions** — only what's needed for deployment
- All traffic is served over **HTTPS** via CloudFront

