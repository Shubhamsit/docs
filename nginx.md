# NGINX on Ubuntu AWS EC2 - Complete Setup Guide

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Installation on Ubuntu](#installation-on-ubuntu)
- [Basic Configuration](#basic-configuration)
- [Securing with SSL/TLS](#securing-with-ssltls)
- [Reverse Proxy Setup](#reverse-proxy-setup)
- [Load Balancing](#load-balancing)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)
- [Real-World Deployment Tips](#real-world-deployment-tips)
- [Topics Covered](#topics-covered)

## Introduction

**NGINX** (pronounced "engine-x") is a high-performance web server, reverse proxy server, and load balancer. Originally created by Igor Sysoev in 2004, NGINX has become one of the most popular web servers worldwide due to its:

- **High Performance**: Handles thousands of concurrent connections with low memory usage
- **Scalability**: Event-driven, asynchronous architecture
- **Flexibility**: Works as web server, reverse proxy, load balancer, and API gateway
- **Reliability**: Powers many of the world's busiest websites

### Why Use NGINX?

- **Web Server**: Serve static content (HTML, CSS, JavaScript, images)
- **Reverse Proxy**: Route requests to backend applications
- **Load Balancer**: Distribute traffic across multiple servers
- **SSL Termination**: Handle SSL/TLS encryption
- **Caching**: Improve performance with content caching
- **Security**: Act as a protective layer for backend services

## Prerequisites

Before starting, ensure you have:

### 1. AWS Account Setup
- Active AWS account with appropriate permissions
- Basic understanding of AWS services (EC2, VPC, Security Groups)

### 2. EC2 Instance Requirements
- **Instance Type**: t2.micro (free tier) or larger
- **Operating System**: Ubuntu 20.04 LTS or 22.04 LTS
- **Storage**: Minimum 8GB (default is sufficient)

### 3. Security Configuration
```bash
# Required Security Group Rules (Inbound)
SSH (22)     - Your IP address
HTTP (80)    - 0.0.0.0/0 (anywhere)
HTTPS (443)  - 0.0.0.0/0 (anywhere)
```

### 4. SSH Key Pair
- Generate or use existing SSH key pair for EC2 access
- Keep the private key (.pem) file secure

### 5. Connecting to EC2 Instance
```bash
# Connect via SSH (replace with your details)
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip

# Example
ssh -i "nginx-server.pem" ubuntu@54.123.456.789
```

> **💡 Pro Tip**: Always update your security groups to allow only necessary traffic. Consider restricting SSH access to your specific IP address for better security.

## Installation on Ubuntu

### Step 1: Update System Packages
```bash
# Update package index
sudo apt update

# Upgrade existing packages (optional but recommended)
sudo apt upgrade -y
```

### Step 2: Install NGINX
```bash
# Install NGINX
sudo apt install nginx -y

# Verify installation
nginx -v
```

### Step 3: Manage NGINX Service
```bash
# Start NGINX
sudo systemctl start nginx

# Enable NGINX to start on boot
sudo systemctl enable nginx

# Check NGINX status
sudo systemctl status nginx

# Other useful commands
sudo systemctl stop nginx     # Stop NGINX
sudo systemctl restart nginx  # Restart NGINX
sudo systemctl reload nginx   # Reload configuration
```

### Step 4: Verify Installation
```bash
# Test if NGINX is running
curl http://localhost

# Check if port 80 is listening
sudo netstat -tlnp | grep :80
```

Visit your EC2 public IP in a browser - you should see the NGINX welcome page.

## Basic Configuration

### Understanding NGINX Directory Structure

```bash
/etc/nginx/                 # Main configuration directory
├── nginx.conf             # Main configuration file
├── sites-available/       # Available site configurations
├── sites-enabled/         # Enabled site configurations (symlinks)
├── conf.d/               # Additional configuration files
└── snippets/             # Configuration snippets
```

### Default Configuration
```nginx
# /etc/nginx/sites-available/default
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;
    
    server_name _;
    
    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Creating a Custom Website

#### 1. Create Website Directory
```bash
# Create directory for your website
sudo mkdir -p /var/www/mywebsite

# Create a simple HTML file
sudo tee /var/www/mywebsite/index.html << EOF
<!DOCTYPE html>
<html>
<head>
    <title>My NGINX Website</title>
</head>
<body>
    <h1>Welcome to My Custom NGINX Site!</h1>
    <p>This website is served by NGINX on AWS EC2.</p>
</body>
</html>
EOF

# Set proper ownership and permissions
sudo chown -R www-data:www-data /var/www/mywebsite
sudo chmod -R 755 /var/www/mywebsite
```

#### 2. Create Server Block Configuration
```bash
# Create new site configuration
sudo nano /etc/nginx/sites-available/mywebsite
```

```nginx
# /etc/nginx/sites-available/mywebsite
server {
    listen 80;
    listen [::]:80;
    
    server_name your-domain.com www.your-domain.com;
    
    root /var/www/mywebsite;
    index index.html index.htm;
    
    location / {
        try_files $uri $uri/ =404;
    }
    
    # Logging
    access_log /var/log/nginx/mywebsite.access.log;
    error_log /var/log/nginx/mywebsite.error.log;
}
```

#### 3. Enable the Site
```bash
# Create symbolic link to enable site
sudo ln -s /etc/nginx/sites-available/mywebsite /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload NGINX
sudo systemctl reload nginx
```

### Managing Multiple Sites
```bash
# Disable default site (optional)
sudo unlink /etc/nginx/sites-enabled/default

# List enabled sites
ls -la /etc/nginx/sites-enabled/

# Enable/disable sites
sudo ln -s /etc/nginx/sites-available/site-name /etc/nginx/sites-enabled/  # Enable
sudo unlink /etc/nginx/sites-enabled/site-name  # Disable
```

### Understanding NGINX Logs
```bash
# Access logs (successful requests)
sudo tail -f /var/log/nginx/access.log

# Error logs (errors and warnings)
sudo tail -f /var/log/nginx/error.log

# Site-specific logs
sudo tail -f /var/log/nginx/mywebsite.access.log
```

## Securing with SSL/TLS

### Why SSL/TLS is Important
- **Data Encryption**: Protects data in transit
- **Authentication**: Verifies server identity
- **SEO Benefits**: Google favors HTTPS sites
- **User Trust**: Browsers mark HTTP sites as "not secure"

### Installing Certbot for Let's Encrypt
```bash
# Install Certbot and NGINX plugin
sudo apt install certbot python3-certbot-nginx -y

# Verify installation
certbot --version
```

### Obtaining SSL Certificate
```bash
# Get certificate for your domain
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Follow the prompts:
# 1. Enter email address
# 2. Agree to terms of service
# 3. Choose whether to share email with EFF
# 4. Choose redirect option (recommended: redirect HTTP to HTTPS)
```

### Manual Certificate Configuration
If you prefer manual configuration:

```bash
# Get certificate only (without auto-configuration)
sudo certbot certonly --nginx -d your-domain.com -d www.your-domain.com
```

```nginx
# Manual HTTPS configuration
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    return 301 https://$server_name$request_uri;  # Redirect to HTTPS
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    
    server_name your-domain.com www.your-domain.com;
    
    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    
    # SSL Security Settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    
    root /var/www/mywebsite;
    index index.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Certificate Auto-Renewal
```bash
# Test auto-renewal
sudo certbot renew --dry-run

# Check renewal timer
sudo systemctl status certbot.timer

# Manual renewal (if needed)
sudo certbot renew
```

> **🔒 Security Note**: Let's Encrypt certificates are valid for 90 days and auto-renew. Always test your renewal process to ensure continuity.

## Reverse Proxy Setup

### What is a Reverse Proxy?

A reverse proxy sits between clients and backend servers, forwarding client requests to backend servers and returning responses back to clients.

### Benefits of Reverse Proxy
- **Security**: Hides backend server details
- **Load Distribution**: Distributes requests across multiple servers
- **SSL Termination**: Handles SSL encryption/decryption
- **Caching**: Improves performance
- **Compression**: Reduces bandwidth usage

### Basic Reverse Proxy Configuration

#### Scenario: Frontend (NGINX) + Backend (Node.js/Python/etc.)

```nginx
# /etc/nginx/sites-available/reverse-proxy
server {
    listen 80;
    server_name your-app.com;
    
    # Proxy to backend application
    location / {
        proxy_pass http://127.0.0.1:3000;  # Backend server
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    # Serve static files directly
    location /static/ {
        alias /var/www/static/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # API endpoints
    location /api/ {
        proxy_pass http://127.0.0.1:3001;  # API server
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # API-specific settings
        proxy_read_timeout 60s;
        proxy_connect_timeout 60s;
    }
}
```

### Advanced Reverse Proxy Configuration

```nginx
# /etc/nginx/sites-available/advanced-proxy
upstream backend {
    server 127.0.0.1:3000 weight=3;
    server 127.0.0.1:3001 weight=2;
    server 127.0.0.1:3002 weight=1;
}

server {
    listen 443 ssl http2;
    server_name your-app.com;
    
    # SSL configuration
    ssl_certificate /etc/letsencrypt/live/your-app.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-app.com/privkey.pem;
    
    location / {
        proxy_pass http://backend;
        
        # Essential proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        # Buffering
        proxy_buffering on;
        proxy_buffer_size 128k;
        proxy_buffers 4 256k;
        proxy_busy_buffers_size 256k;
        
        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

## Load Balancing

### Load Balancing Methods

#### 1. Round Robin (Default)
```nginx
upstream backend {
    server 10.0.1.10:3000;
    server 10.0.1.11:3000;
    server 10.0.1.12:3000;
}
```

#### 2. Weighted Round Robin
```nginx
upstream backend {
    server 10.0.1.10:3000 weight=3;  # Receives 3x more requests
    server 10.0.1.11:3000 weight=2;  # Receives 2x more requests
    server 10.0.1.12:3000 weight=1;  # Receives 1x requests
}
```

#### 3. Least Connections
```nginx
upstream backend {
    least_conn;
    server 10.0.1.10:3000;
    server 10.0.1.11:3000;
    server 10.0.1.12:3000;
}
```

#### 4. IP Hash (Session Persistence)
```nginx
upstream backend {
    ip_hash;
    server 10.0.1.10:3000;
    server 10.0.1.11:3000;
    server 10.0.1.12:3000;
}
```

### Health Checks and Failover

```nginx
upstream backend {
    server 10.0.1.10:3000 max_fails=3 fail_timeout=30s;
    server 10.0.1.11:3000 max_fails=3 fail_timeout=30s;
    server 10.0.1.12:3000 backup;  # Backup server
}

server {
    listen 80;
    server_name app.example.com;
    
    location / {
        proxy_pass http://backend;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
        
        # Health check
        proxy_connect_timeout 5s;
        proxy_send_timeout 10s;
        proxy_read_timeout 10s;
    }
}
```

### Multi-Tier Load Balancing

```nginx
# Database tier
upstream db_servers {
    server db1.internal:5432 weight=2;
    server db2.internal:5432 weight=1;
    server db3.internal:5432 backup;
}

# Application tier
upstream app_servers {
    least_conn;
    server app1.internal:3000 max_fails=3 fail_timeout=30s;
    server app2.internal:3000 max_fails=3 fail_timeout=30s;
    server app3.internal:3000 max_fails=3 fail_timeout=30s;
}

# Web tier
server {
    listen 443 ssl http2;
    server_name myapp.com;
    
    location /api/db/ {
        proxy_pass http://db_servers;
    }
    
    location /api/ {
        proxy_pass http://app_servers;
    }
    
    location / {
        root /var/www/html;
        try_files $uri $uri/ /index.html;
    }
}
```

## Advanced Features

### Custom Error Pages

```nginx
server {
    listen 80;
    server_name example.com;
    
    # Custom error pages
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    
    location = /404.html {
        root /var/www/errors;
        internal;
    }
    
    location = /50x.html {
        root /var/www/errors;
        internal;
    }
}
```

Create custom error pages:
```bash
# Create error pages directory
sudo mkdir -p /var/www/errors

# Create 404 page
sudo tee /var/www/errors/404.html << EOF
<!DOCTYPE html>
<html>
<head><title>Page Not Found</title></head>
<body>
    <h1>404 - Page Not Found</h1>
    <p>Sorry, the page you're looking for doesn't exist.</p>
</body>
</html>
EOF
```

### Caching Configuration

```nginx
# Define cache paths in main nginx.conf
http {
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=10g 
                     inactive=60m use_temp_path=off;
}

server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://backend;
        
        # Caching
        proxy_cache my_cache;
        proxy_cache_revalidate on;
        proxy_cache_min_uses 1;
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
        proxy_cache_lock on;
        
        # Cache based on these variables
        proxy_cache_key "$scheme$request_method$host$request_uri";
        proxy_cache_valid 200 304 12h;
        proxy_cache_valid 404 1m;
        
        # Add cache status header
        add_header X-Cache-Status $upstream_cache_status;
    }
    
    # Don't cache API calls
    location /api/ {
        proxy_pass http://backend;
        proxy_cache_bypass 1;
        proxy_no_cache 1;
    }
}
```

### Rate Limiting

```nginx
# Define rate limiting zones in main nginx.conf
http {
    limit_req_zone $binary_remote_addr zone=login:10m rate=1r/s;
    limit_req_zone $binary_remote_addr zone=general:10m rate=10r/s;
}

server {
    listen 80;
    server_name example.com;
    
    # General rate limiting
    location / {
        limit_req zone=general burst=20 nodelay;
        proxy_pass http://backend;
    }
    
    # Strict rate limiting for login
    location /login {
        limit_req zone=login burst=5;
        proxy_pass http://backend;
    }
}
```

### Gzip Compression

```nginx
server {
    listen 80;
    server_name example.com;
    
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;
    
    location / {
        proxy_pass http://backend;
    }
}
```

## Best Practices

### 1. Configuration Testing

```bash
# Always test configuration before applying
sudo nginx -t

# Test specific configuration file
sudo nginx -t -c /etc/nginx/nginx.conf

# Check configuration syntax only
sudo nginx -T
```

> **⚠️ Critical**: Never reload NGINX without testing configuration first. A syntax error can bring down your entire web service.

### 2. Safe Configuration Reloading

```bash
# Test first, then reload
sudo nginx -t && sudo systemctl reload nginx

# Or use this one-liner for safety
test $(sudo nginx -t 2>&1 | grep -c "syntax is ok") -eq 1 && sudo systemctl reload nginx || echo "Configuration has errors!"
```

### 3. Monitoring and Logging

```nginx
# Enhanced logging format
log_format detailed '$remote_addr - $remote_user [$time_local] '
                   '"$request" $status $bytes_sent '
                   '"$http_referer" "$http_user_agent" '
                   '$request_time $upstream_response_time';

server {
    access_log /var/log/nginx/access.log detailed;
    error_log /var/log/nginx/error.log warn;
}
```

```bash
# Monitor logs in real-time
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# Analyze access patterns
sudo awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Check for errors
sudo grep "error" /var/log/nginx/error.log | tail -20
```

### 4. Security Headers

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';" always;
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    
    location / {
        proxy_pass http://backend;
    }
}
```

### 5. Performance Optimization

```nginx
# In main nginx.conf
worker_processes auto;
worker_connections 1024;

http {
    # Optimize file operations
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    
    # Timeouts
    keepalive_timeout 65;
    client_body_timeout 12;
    client_header_timeout 12;
    send_timeout 10;
    
    # Buffer sizes
    client_body_buffer_size 128k;
    client_max_body_size 10m;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 4k;
}
```

### 6. Backup Configuration

```bash
# Create configuration backup
sudo cp -r /etc/nginx /etc/nginx-backup-$(date +%Y%m%d)

# Or create a script for regular backups
sudo tee /usr/local/bin/backup-nginx.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/var/backups/nginx"
DATE=$(date +%Y%m%d-%H%M%S)

mkdir -p $BACKUP_DIR
tar -czf $BACKUP_DIR/nginx-config-$DATE.tar.gz /etc/nginx
echo "NGINX configuration backed up to $BACKUP_DIR/nginx-config-$DATE.tar.gz"
EOF

sudo chmod +x /usr/local/bin/backup-nginx.sh
```

## Real-World Deployment Tips

### 1. EC2 Security Group Configuration

```bash
# Recommended Security Group Rules
Port 22 (SSH)    - Your IP only (not 0.0.0.0/0)
Port 80 (HTTP)   - 0.0.0.0/0 (for Let's Encrypt validation)
Port 443 (HTTPS) - 0.0.0.0/0
Port 3000-3010   - Internal VPC only (for backend services)
```

### 2. Domain Setup with Route 53

```bash
# Create hosted zone in Route 53
# Add these records:
A Record:     @ -> EC2_PUBLIC_IP
A Record:     www -> EC2_PUBLIC_IP
CNAME Record: * -> example.com (for subdomains)
```

### 3. Auto-Scaling Architecture

```nginx
# Use Application Load Balancer (ALB) + Auto Scaling Group
# NGINX configuration for health checks
location /health {
    access_log off;
    return 200 "healthy\n";
    add_header Content-Type text/plain;
}
```

### 4. Multi-AZ Deployment

```bash
# Deploy NGINX in multiple availability zones
# Use Amazon EFS for shared static content
# Example mount for shared content:
sudo mount -t efs fs-xxxxxx.efs.region.amazonaws.com:/ /var/www/shared
```

### 5. Monitoring and Alerting

```bash
# Install CloudWatch agent for advanced monitoring
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i amazon-cloudwatch-agent.deb

# Monitor NGINX metrics
sudo apt install nginx-module-prometheus -y
```

### 6. SSL Certificate Management at Scale

```bash
# Use AWS Certificate Manager (ACM) with ALB
# Or automated Let's Encrypt with DNS validation
sudo certbot certonly --dns-route53 -d *.example.com -d example.com
```

### 7. Disaster Recovery

```bash
# Automated backup script for EC2
#!/bin/bash
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
aws ec2 create-snapshot --volume-id vol-xxxxxx --description "NGINX-backup-$(date +%Y%m%d)"
```

### 8. Performance Monitoring

```nginx
# Enable NGINX status page
location /nginx_status {
    stub_status on;
    access_log off;
    allow 127.0.0.1;
    deny all;
}
```

```bash
# Monitor with curl
curl http://localhost/nginx_status
```

> **🚀 Pro Deployment Tip**: Always use Infrastructure as Code (Terraform/CloudFormation) for production deployments. This ensures consistency and makes disaster recovery much easier.

## Topics Covered

This comprehensive guide covered the following topics:

### **Introduction & Fundamentals**
- What is NGINX and its core benefits
- Use cases: web server, reverse proxy, load balancer
- Performance advantages and architecture

### **Prerequisites & Setup**
- AWS account and EC2 instance setup
- Ubuntu system requirements
- SSH key pair management and connection
- Security group configuration for HTTP/HTTPS traffic

### **Installation & Basic Configuration**
- Ubuntu package management and NGINX installation
- Service management (start, stop, enable, status)
- Directory structure understanding (/etc/nginx/, sites-available/, sites-enabled/)
- Creating custom websites and server blocks
- Log file management and monitoring

### **SSL/TLS Security Implementation**
- Let's Encrypt certificate installation with Certbot
- Automatic HTTPS configuration and HTTP-to-HTTPS redirection
- Manual SSL certificate configuration
- Security headers and best practices
- Certificate auto-renewal setup

### **Reverse Proxy Configuration**
- Understanding reverse proxy concepts and benefits
- Frontend-backend separation architecture
- Proxy headers and security considerations
- Static file serving optimization
- API endpoint routing

### **Load Balancing Strategies**
- Round-robin, weighted round-robin, and least connections
- IP hash for session persistence
- Health checks and failover mechanisms
- Multi-tier load balancing architecture
- Upstream server configuration

### **Advanced Features**
- Custom error pages creation and configuration
- Caching strategies and proxy cache setup
- Rate limiting for security and performance
- Gzip compression for bandwidth optimization
- WebSocket support configuration

### **Best Practices & Operations**
- Configuration testing with `nginx -t`
- Safe reloading procedures
- Enhanced logging and monitoring
- Security headers implementation
- Performance optimization techniques
- Configuration backup strategies

### **Real-World AWS Deployment**
- EC2 security group optimization
- Route 53 domain configuration
- Auto-scaling architecture patterns
- Multi-AZ deployment strategies
- CloudWatch monitoring integration
- SSL certificate management at scale
- Disaster recovery planning

This guide provides a complete foundation for deploying, configuring, and managing NGINX on Ubuntu within AWS EC2 instances, from basic installation to enterprise-level production deployments.