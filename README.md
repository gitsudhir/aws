# AWS EC2 Connection Guide

## Instance Details
- Instance ID: i-00d8a7ab13c96d8c2 (aws-ec2-sudhir)
- Public IP: 18.60.184.7
- Private IP: 172.31.11.186
- OS: Ubuntu
- SSH User: ubuntu

## Prerequisites
- AWS EC2 private key file (aws-ec2-key-sk.pem)
- SSH client installed on your local machine

## Connecting to Your EC2 Instance

### 1. Set Proper Permissions on Private Key
Before connecting, ensure your private key has the correct permissions:

```bash
chmod 400 aws-ec2-key-sk.pem
```

### 2. Connect Using SSH
Use the following command to connect to your EC2 instance:

```bash
ssh -i aws-ec2-key-sk.pem ubuntu@18.60.184.7
```

Alternatively, if you're connecting from within the same VPC or have VPN access:

```bash
ssh -i aws-ec2-key-sk.pem ubuntu@172.31.11.186
```

### 3. Connect with X11 Forwarding for GUI Applications
To use GUI applications like gedit, connect with X11 forwarding enabled:

```bash
ssh -X -i aws-ec2-key-sk.pem ubuntu@18.60.184.7
```

Note: You'll need an X11 server installed on your local machine:
- On macOS: Install XQuartz using Homebrew:
  ```bash
  brew install --cask xquartz
  ```
  Or download from https://www.xquartz.org/
- On Windows: Install an X server like Xming or use WSL2 with WSLg

After installing XQuartz on macOS, restart your Mac or log out and back in, then start XQuartz from your Applications folder.

After connecting with X11 forwarding, you can install and use GUI applications:

```bash
# Install GUI applications
sudo apt update
sudo apt install gedit

# Launch GUI applications
gedit filename.txt
```

## Security Notes
- Never share your private key file
- Never commit private keys to version control
- Regularly rotate your SSH keys
- Consider using AWS Systems Manager Session Manager for keyless access

## Setting Up a Web Server (Nginx)

If you're seeing the message **"Unit nginx.service could not be found"**, it means **Nginx is NOT installed** on your EC2 instance.

Your website cannot load because **no web server is running**.

### ✅ Fix the "site can't be reached" issue

You need to install a web server such as **Nginx** or **Apache** (Nginx recommended).

Below are the steps.

### ⭐ STEP 1 — Install Nginx

Run:

```bash
sudo apt update
sudo apt install nginx -y
```

### ⭐ STEP 2 — Start Nginx

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

Check status:

```bash
sudo systemctl status nginx
```

Should show: **active (running)**

### ⭐ STEP 3 — Open Ports 80 and 443 in Security Group

Go to:

EC2 Console → Instances → Select your instance
→ Security → **Click Security Group** → Inbound rules

Add/verify these rules:

| Type  | Port | Source    |
| ----- | ---- | --------- |
| HTTP  | 80   | 0.0.0.0/0 |
| HTTPS | 443  | 0.0.0.0/0 |

### ⭐ STEP 4 — Use the NEW PUBLIC IP

Since your instance uses auto-assigned IP, check:

EC2 → Instances → **Public IPv4**

It will look something like:
**18.xx.xx.xx**

Open this in browser:

```
http://YOUR_PUBLIC_IP
```

You should see:

### **Welcome to Nginx!**

## ✅ Nginx Successfully Installed

Congratulations! If you're seeing the "Welcome to Nginx!" page at http://18.60.184.7, your web server is successfully installed and running.

## 🔄 Replacing the Default Nginx Page with Your Custom HTML

Perfect! You already have your new HTML file at:

```
/home/ubuntu/sudhir/sudhirkumar.html
```

To make **Nginx serve this page** instead of the default, follow these steps:

### ✅ Step 1 — Backup the existing default page

```bash
sudo mv /var/www/html/index.nginx-debian.html /var/www/html/index.nginx-debian.html.bak
```

This keeps a backup in case you want the default page later.

### ✅ Step 2 — Copy your new HTML file to Nginx root

```bash
sudo cp /home/ubuntu/sudhir/sudhirkumar.html /var/www/html/index.html
```

* Nginx looks for **`/var/www/html/index.html`** by default.
* We renamed your file to `index.html` for Nginx to serve automatically.

### ✅ Step 3 — Set correct permissions

```bash
sudo chown www-data:www-data /var/www/html/index.html
sudo chmod 644 /var/www/html/index.html
```

This ensures Nginx can read the file.

### ✅ Step 4 — Restart Nginx

```bash
sudo systemctl restart nginx
```

### ✅ Step 5 — Test in browser

Open:

```
http://YOUR_PUBLIC_IP
```

You should now see your **`sudhirkumar.html`** page served by Nginx.

### Next Steps for Website Configuration

1. **Place your website files** in the default directory:
   ```bash
   sudo nano /var/www/html/index.html
   ```

2. **Create a custom server block** for your domain:
   ```bash
   sudo nano /etc/nginx/sites-available/your-domain
   ```
   
   Add configuration:
   ```nginx
   server {
       listen 80;
       server_name your-domain.com;
       root /var/www/your-domain;
       index index.html index.htm;
       
       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```

3. **Enable your site**:
   ```bash
   sudo ln -s /etc/nginx/sites-available/your-domain /etc/nginx/sites-enabled/
   sudo nginx -t  # Test configuration
   sudo systemctl reload nginx  # Reload Nginx
   ```

4. **Host Your Portfolio Website** (sudhirkumar.in):
   To replace the default Nginx welcome page with your portfolio:
   
   a. **Create a directory for your website**:
   ```bash
   sudo mkdir -p /var/www/sudhirkumar.in/html
   sudo chown -R $USER:$USER /var/www/sudhirkumar.in/html
   sudo chmod -R 755 /var/www/sudhirkumar.in
   ```
   
   b. **Create a simple index.html or copy your portfolio files**:
   ```bash
   nano /var/www/sudhirkumar.in/html/index.html
   ```
   
   c. **Create a server block for your domain**:
   ```bash
   sudo nano /etc/nginx/sites-available/sudhirkumar.in
   ```
   
   Add the following configuration:
   ```nginx
   server {
       listen 80;
       listen [::]:80;
       
       root /var/www/sudhirkumar.in/html;
       index index.html index.htm index.nginx-debian.html;
       
       server_name sudhirkumar.in www.sudhirkumar.in;
       
       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```
   
   d. **Enable the server block**:
   ```bash
   sudo ln -s /etc/nginx/sites-available/sudhirkumar.in /etc/nginx/sites-enabled/
   ```
   
   e. **Test and reload Nginx**:
   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

5. **Set up SSL** (recommended):
   Use Let's Encrypt with Certbot for free SSL certificates.

## 🔒 Setting Up SSL with Let's Encrypt (Certbot)

SSL encryption is essential for securing your website and protecting user data. Let's Encrypt provides free SSL certificates that are trusted by all major browsers.

### ⭐ Prerequisites

Before setting up SSL, ensure you have:
- Your domain name (sudhirkumar.in) pointing to your EC2 instance's public IP (18.60.184.7)
- Nginx properly configured with your domain
- Port 443 open in your security group

### ⭐ Installation Steps

1. **Install Certbot and the Nginx plugin**:
   ```bash
   sudo apt update
   sudo apt install certbot python3-certbot-nginx -y
   ```

2. **Obtain and install the certificate**:
   ```bash
   sudo certbot --nginx -d sudhirkumar.in -d www.sudhirkumar.in
   ```
   
3. **Follow the prompts**:
   - Enter your email for important notifications
   - Agree to the terms of service
   - Choose whether to redirect HTTP traffic to HTTPS (recommended: Yes)

### ⭐ Automatic Renewal

Let's Encrypt certificates expire every 90 days. Set up automatic renewal:

1. **Test automatic renewal**:
   ```bash
   sudo certbot renew --dry-run
   ```

2. **The certbot package automatically creates a cron job** for renewal, but you can verify it exists:
   ```bash
   sudo systemctl status certbot.timer
   ```

### ⭐ Manual Certificate Management

- **List certificates**: `sudo certbot certificates`
- **Revoke a certificate**: `sudo certbot revoke --cert-path /etc/letsencrypt/live/sudhirkumar.in/fullchain.pem`
- **Delete a certificate**: `sudo certbot delete --cert-name sudhirkumar.in`

### ⭐ Troubleshooting

If you encounter issues:
- Ensure your domain (sudhirkumar.in) points to the correct IP address (18.60.184.7)
- Verify ports 80 and 443 are accessible (required for HTTP challenge)
- Check Nginx configuration: `sudo nginx -t`
- Review Certbot logs: `sudo tail -f /var/log/letsencrypt/letsencrypt.log

## 📦 Integrating with Amazon S3

Amazon S3 (Simple Storage Service) is an object storage service that offers industry-leading scalability, data availability, security, and performance. You can use S3 to store static assets for your website, backups, or as a data lake for analytics.

### ⭐ Setting Up S3 Bucket for Static Website Hosting

1. **Create an S3 bucket**:
   ```bash
   # Install AWS CLI if not already installed
   sudo apt update
   sudo apt install awscli -y
   
   # Configure AWS CLI with your credentials
   aws configure
   # Enter your AWS Access Key ID, Secret Access Key, region (us-east-1), and output format (json)
   
   # Create S3 bucket (replace 'your-unique-bucket-name' with a globally unique name)
   aws s3 mb s3://your-unique-bucket-name
   
   # Enable static website hosting
   aws s3 website s3://your-unique-bucket-name --index-document index.html --error-document error.html
   ```

2. **Upload your website files**:
   ```bash
   # Upload files to your S3 bucket
   aws s3 cp /var/www/html/ s3://your-unique-bucket-name --recursive
   
   # Set public read permissions
   aws s3api put-bucket-policy --bucket your-unique-bucket-name --policy '{
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::your-unique-bucket-name/*"
       }
     ]
   }'
   ```

3. **Access your website**:
   Your website will be available at:
   ```
   http://your-unique-bucket-name.s3-website-us-east-1.amazonaws.com
   ```

### ⭐ Using S3 for Media Assets with Nginx

You can configure Nginx to serve media assets directly from S3 while keeping your main application on EC2:

1. **Modify your Nginx configuration**:
   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```
   
   Add a location block for media assets:
   ```nginx
   location /media/ {
       proxy_pass https://your-s3-bucket.s3.amazonaws.com/;
       proxy_set_header Host your-s3-bucket.s3.amazonaws.com;
   }
   ```

2. **Test and reload Nginx**:
   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

### ⭐ S3 Backup Strategy

Set up automated backups of your EC2 instance data to S3:

1. **Create a backup script**:
   ```bash
   nano ~/backup-to-s3.sh
   ```
   
   Add the following content:
   ```bash
   #!/bin/bash
   DATE=$(date +%Y-%m-%d-%H-%M-%S)
   BACKUP_NAME="backup-$DATE.tar.gz"
   
   # Create archive of important directories
   tar -czf /tmp/$BACKUP_NAME /var/www/html /etc/nginx
   
   # Upload to S3
   aws s3 cp /tmp/$BACKUP_NAME s3://your-backup-bucket/backups/
   
   # Remove local backup file
   rm /tmp/$BACKUP_NAME
   
   echo "Backup completed: $BACKUP_NAME"
   ```

2. **Make the script executable and schedule it**:
   ```bash
   chmod +x ~/backup-to-s3.sh
   
   # Add to crontab to run daily at 2 AM
   (crontab -l 2>/dev/null; echo "0 2 * * * /home/ubuntu/backup-to-s3.sh") | crontab -
   ```

### ⭐ Security Best Practices for S3

1. **Block public access** unless absolutely necessary:
   ```bash
   aws s3api put-public-access-block --bucket your-bucket-name --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
   ```

2. **Enable server-side encryption**:
   ```bash
   aws s3api put-bucket-encryption --bucket your-bucket-name --server-side-encryption-configuration '{
     "Rules": [
       {
         "ApplyServerSideEncryptionByDefault": {
           "SSEAlgorithm": "AES256"
         }
       }
     ]
   }'
   ```

3. **Enable versioning for data protection**:
   ```bash
   aws s3api put-bucket-versioning --bucket your-bucket-name --versioning-configuration Status=Enabled
   ```

4. **Enable logging**:
   ```bash
   aws s3api put-bucket-logging --bucket your-bucket-name --bucket-logging-status '{
     "LoggingEnabled": {
       "TargetBucket": "your-log-bucket",
       "TargetPrefix": "logs/"
     }
   }'
   ```