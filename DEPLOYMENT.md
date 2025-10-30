# GitHub Actions EC2 Deployment Setup

This repository includes a GitHub Actions workflow that automatically deploys your DevOps roadmap website to an EC2 instance whenever you push to the main branch.

## Prerequisites

Before the deployment workflow can run successfully, you need to set up the following:

### 1. EC2 Instance Setup

Your EC2 instance should have:

- **Ubuntu Server** (latest LTS recommended)
- **Nginx** web server installed and configured
- **Git** installed
- **SSH access** enabled
- **Security Group** allowing HTTP (port 80) and SSH (port 22) traffic

#### Quick EC2 Setup Commands:

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Nginx
sudo apt install nginx -y

# Install Git
sudo apt install git -y

# Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Clone your repository to web directory
sudo rm -rf /var/www/html/*
sudo git clone https://github.com/YOUR_USERNAME/my-devops-app.git /var/www/html
cd /var/www/html

# Set proper permissions
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html

# Configure Git for pulls (run as www-data user)
sudo -u www-data git config --global --add safe.directory /var/www/html
```

### 2. GitHub Repository Secrets

You need to add the following secrets to your GitHub repository:

#### Navigate to: Repository Settings → Secrets and variables → Actions → New repository secret

**Required Secrets:**

1. **`EC2_HOST`**
   - **Value**: Your EC2 instance's public IP address or domain name
   - **Example**: `3.15.123.456` or `your-domain.com`

2. **`EC2_SSH_KEY`**
   - **Value**: Your private SSH key content (the entire key file)
   - **Format**: The content of your `~/.ssh/id_ed25519` or `~/.ssh/id_rsa` file

#### How to get your SSH key:

```bash
# If you already have an SSH key
cat ~/.ssh/id_ed25519

# Or if using RSA key
cat ~/.ssh/id_rsa

# Copy the ENTIRE output including the BEGIN/END lines
```

#### How to add your public key to EC2:

```bash
# On your EC2 instance, add your public key to authorized_keys
echo "YOUR_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### 3. Nginx Configuration (Optional)

For better performance, you can create a custom Nginx configuration:

```bash
# Create Nginx site configuration
sudo nano /etc/nginx/sites-available/devops-roadmap
```

Add this configuration:

```nginx
server {
    listen 80;
    server_name your-domain.com;  # Replace with your domain or IP
    root /var/www/html;
    index index.html;

    # Enable gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # Cache static assets
    location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Main location block
    location / {
        try_files $uri $uri/ =404;
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
}
```

Enable the site:

```bash
# Enable the site
sudo ln -s /etc/nginx/sites-available/devops-roadmap /etc/nginx/sites-enabled/

# Remove default site (optional)
sudo rm /etc/nginx/sites-enabled/default

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

## Workflow Features

The GitHub Actions workflow includes:

### ✅ **Automated Testing**
- HTML validation using `tidy`
- CSS syntax checking
- Runs on every push and pull request

### ✅ **Safe Deployment**
- Only deploys from `main` branch
- Creates backup before deployment
- Tests Nginx configuration
- Verifies website is responding after deployment

### ✅ **Error Handling**
- Timeout protection (60 seconds)
- Rollback capability with backups
- Status notifications

### ✅ **Security Best Practices**
- Uses SSH key authentication
- Proper file permissions
- Configuration validation

## Workflow Triggers

The deployment runs automatically when:

1. **Push to main branch** → Runs tests + deployment
2. **Pull request to main** → Runs tests only (no deployment)

## Manual Deployment

If you need to deploy manually:

```bash
# SSH into your EC2 instance
ssh ubuntu@YOUR_EC2_IP

# Navigate to web directory
cd /var/www/html

# Pull latest changes
sudo git pull origin main

# Set permissions
sudo chown -R www-data:www-data .

# Reload Nginx
sudo systemctl reload nginx
```

## Troubleshooting

### Common Issues:

1. **SSH Connection Failed**
   - Verify EC2 security group allows SSH (port 22)
   - Check that `EC2_HOST` secret contains correct IP/domain
   - Ensure `EC2_SSH_KEY` secret contains the complete private key

2. **Git Pull Failed**
   - Check that the repository is properly cloned in `/var/www/html`
   - Verify Git configuration on EC2 instance
   - Ensure proper permissions for www-data user

3. **Nginx Errors**
   - Check Nginx logs: `sudo tail -f /var/log/nginx/error.log`
   - Verify Nginx configuration: `sudo nginx -t`
   - Check file permissions in `/var/www/html`

4. **Website Not Loading**
   - Verify EC2 security group allows HTTP (port 80)
   - Check if Nginx is running: `sudo systemctl status nginx`
   - Verify DNS configuration if using a domain name

### Debug Commands:

```bash
# Check deployment status
systemctl status nginx
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# Verify file permissions
ls -la /var/www/html/

# Test website locally on EC2
curl http://localhost/

# Check Git status
cd /var/www/html && git status
```

## Security Recommendations

1. **Use SSH keys instead of passwords**
2. **Regularly update your EC2 instance**
3. **Configure a firewall (UFW)**
4. **Use HTTPS with SSL certificates (Let's Encrypt)**
5. **Regularly rotate SSH keys**
6. **Monitor deployment logs**

## Next Steps

After setting up the deployment:

1. **Set up monitoring** with CloudWatch or other tools
2. **Configure SSL/HTTPS** with Let's Encrypt
3. **Set up a custom domain name**
4. **Add automated backups**
5. **Implement blue-green deployment** for zero downtime

Your DevOps roadmap website will now automatically deploy to EC2 every time you push changes to the main branch! 🚀