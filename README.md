# Sudoku Launchpad — Homepage

Product landing page for [Sudoku Launchpad](https://sudokulaunchpad.com), a companion app for sudoku enthusiasts who solve on SudokuPad and follow creators like Cracking The Cryptic.

## What is Sudoku Launchpad?

Sudoku Launchpad syncs your puzzle progress from SudokuPad, lets you replay your solves move-by-move, and watch them synchronized alongside the original YouTube videos. It also aggregates daily puzzles from the NYT and featured puzzles from CTC into one place.

## Tech Stack

Static single-page site — no build step, no dependencies.

- **HTML** with embedded CSS and JavaScript
- **Light/dark mode** via `data-theme` attribute with system preference detection
- **Theme-aware screenshots** — light and dark image variants swap automatically on theme toggle

## Local Development

Serve the directory with any static file server:

```sh
python3 -m http.server 8080
```

Then open `http://localhost:8080`.

## Screenshot Placeholders

The `images/` directory contains SVG placeholders for app screenshots. Each has a light and dark variant:

| Section | Light | Dark |
|---|---|---|
| Homepage hero | `homepage-light.svg` | `homepage-dark.svg` |
| Video Replay | `replay-video-light.svg` | `replay-video-dark.svg` |
| Calendar | `calendar-light.svg` | `calendar-dark.svg` |
| Statistics | `stats-light.svg` | `stats-dark.svg` |

Replace these with actual screenshots (`.png` or `.webp`) and update the `src` attributes in `index.html`.

## Hosting on AWS

This site is designed to be hosted on **S3 + CloudFront** with DNS managed by **Route 53**.

### 1. Create an S3 Bucket

```sh
aws s3 mb s3://sudokulaunchpad.com
```

Upload the site files:

```sh
aws s3 sync . s3://sudokulaunchpad.com \
  --exclude ".git/*" \
  --exclude ".claude/*" \
  --exclude ".gitignore" \
  --exclude "README.md"
```

The bucket does **not** need static website hosting enabled — CloudFront will serve the files directly via Origin Access Control (OAC).

### 2. Request an SSL Certificate (ACM)

Request a certificate in **us-east-1** (required for CloudFront):

```sh
aws acm request-certificate \
  --domain-name sudokulaunchpad.com \
  --subject-alternative-names "*.sudokulaunchpad.com" \
  --validation-method DNS \
  --region us-east-1
```

ACM will provide a CNAME record for validation. Add it in Route 53 (or your registrar) and wait for the certificate status to become `ISSUED`.

### 3. Create a CloudFront Distribution

Create a distribution with:

- **Origin**: the S3 bucket (use the bucket's S3 REST endpoint, not the website endpoint)
- **Origin Access Control (OAC)**: create one and attach it so CloudFront can read from the private bucket
- **Alternate domain names (CNAMEs)**: `sudokulaunchpad.com`, `www.sudokulaunchpad.com`
- **SSL certificate**: select the ACM certificate from step 2
- **Default root object**: `index.html`
- **Custom error response**: map 404 errors to `/404.html` with a 404 status code
- **CloudFront Function** (`sudokulaunchpad-www-redirect`): attached to viewer-request, 301 redirects `www.sudokulaunchpad.com` to `sudokulaunchpad.com`

After creating the distribution, update the S3 bucket policy to allow CloudFront access:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontOAC",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::sudokulaunchpad.com/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::277389144278:distribution/E9QUSUAFCAG0Y"
        }
      }
    }
  ]
}
```

### 4. Configure Route 53 DNS

The domain is registered in Route 53. The hosted zone contains:

- **A record** — `sudokulaunchpad.com` → Alias to the CloudFront distribution
- **AAAA record** — `sudokulaunchpad.com` → Alias to the CloudFront distribution (for IPv6)
- **A record** — `www.sudokulaunchpad.com` → Alias to the CloudFront distribution
- **AAAA record** — `www.sudokulaunchpad.com` → Alias to the CloudFront distribution (for IPv6)

### 5. Deploy Updates

After making changes, sync to S3 and invalidate the CloudFront cache:

```sh
aws s3 sync . s3://sudokulaunchpad.com \
  --exclude ".git/*" \
  --exclude ".claude/*" \
  --exclude ".gitignore" \
  --exclude "README.md"

aws cloudfront create-invalidation \
  --distribution-id E9QUSUAFCAG0Y \
  --paths "/*"
```

## Project Structure

```
├── index.html          # Landing page (HTML, CSS, JS all-in-one)
├── 404.html            # Custom error page
├── favicon.svg         # SVG favicon
├── robots.txt          # Robots config
└── images/             # Screenshot placeholders (light + dark)
```
