# SreyasCore Website Deployment Guide

This guide will help you deploy the SreyasCore website to sreyascore.net.

## Prerequisites

- Domain name: `sreyascore.net`
- Web hosting service (shared hosting, VPS, or cloud platform)
- FTP/SFTP access or Git deployment capability
- Basic knowledge of web hosting

## Deployment Options

### Option 1: Traditional Web Hosting (Recommended for beginners)

#### Step 1: Choose a Web Host
Recommended providers:
- **Bluehost** - Good for beginners, includes domain
- **HostGator** - Reliable shared hosting
- **SiteGround** - Excellent performance
- **A2 Hosting** - Fast and developer-friendly

#### Step 2: Upload Files
1. **Via FTP/SFTP:**
   - Use FileZilla or WinSCP
   - Connect to your hosting server
   - Upload all files from `website/` folder to `public_html/` or `www/`

2. **Via cPanel File Manager:**
   - Log into your hosting control panel
   - Open File Manager
   - Navigate to `public_html/`
   - Upload and extract the website files

#### Step 3: Configure Domain
1. Point your domain to your hosting provider's nameservers
2. Wait for DNS propagation (24-48 hours)
3. Test the website at `sreyascore.net`

### Option 2: Cloud Hosting (Advanced users)

#### GitHub Pages + Custom Domain
1. **Create GitHub Repository:**
   ```bash
   git init
   git add .
   git commit -m "Initial SreyasCore website"
   git remote add origin https://github.com/yourusername/sreyascore-website.git
   git push -u origin main
   ```

2. **Enable GitHub Pages:**
   - Go to repository Settings → Pages
   - Select source branch (main)
   - Set custom domain to `sreyascore.net`
   - Add CNAME record in your domain provider

3. **Configure DNS:**
   ```
   Type: CNAME
   Name: @
   Value: yourusername.github.io
   ```

#### Netlify Deployment
1. **Connect Repository:**
   - Sign up at netlify.com
   - Connect your GitHub repository
   - Set build command: (leave empty for static site)
   - Set publish directory: `website/`

2. **Custom Domain:**
   - Add custom domain: `sreyascore.net`
   - Follow DNS configuration instructions

#### Vercel Deployment
1. **Import Project:**
   - Sign up at vercel.com
   - Import your GitHub repository
   - Vercel will auto-detect it's a static site

2. **Configure Domain:**
   - Add custom domain: `sreyascore.net`
   - Update DNS records as instructed

### Option 3: VPS/Cloud Server (Full control)

#### DigitalOcean Droplet
1. **Create Droplet:**
   ```bash
   # Ubuntu 22.04 LTS
   # Basic plan: $6/month
   # Add SSH key for security
   ```

2. **Install Web Server:**
   ```bash
   sudo apt update
   sudo apt install nginx
   sudo systemctl enable nginx
   sudo systemctl start nginx
   ```

3. **Upload Website:**
   ```bash
   # Create website directory
   sudo mkdir -p /var/www/sreyascore.net
   sudo chown $USER:$USER /var/www/sreyascore.net
   
   # Upload files via SCP or Git
   scp -r website/* user@your-server:/var/www/sreyascore.net/
   ```

4. **Configure Nginx:**
   ```nginx
   server {
       listen 80;
       server_name sreyascore.net www.sreyascore.net;
       root /var/www/sreyascore.net;
       index index.html;
       
       location / {
           try_files $uri $uri/ =404;
       }
       
       # Enable gzip compression
       gzip on;
       gzip_types text/plain text/css application/json application/javascript;
   }
   ```

5. **Enable HTTPS:**
   ```bash
   sudo apt install certbot python3-certbot-nginx
   sudo certbot --nginx -d sreyascore.net -d www.sreyascore.net
   ```

### Option 4: Advanced Cloud Platforms

#### AWS S3 + CloudFront
1. **Create S3 Bucket:**
   ```bash
   aws s3 mb s3://sreyascore.net
   aws s3 website s3://sreyascore.net --index-document index.html --error-document 404.html
   ```

2. **Upload Website:**
   ```bash
   aws s3 sync website/ s3://sreyascore.net --delete
   ```

3. **Configure CloudFront:**
   - Create distribution pointing to S3 bucket
   - Set custom domain and SSL certificate
   - Configure caching rules

#### Google Cloud Platform
1. **Create Storage Bucket:**
   ```bash
   gsutil mb gs://sreyascore.net
   gsutil website set -m index.html -e 404.html gs://sreyascore.net
   ```

2. **Upload and Configure:**
   ```bash
   gsutil -m rsync -r website/ gs://sreyascore.net/
   gsutil iam ch allUsers:objectViewer gs://sreyascore.net
   ```

#### Microsoft Azure
1. **Create Storage Account:**
   ```bash
   az storage account create --name sreyascore --resource-group myResourceGroup --location eastus --sku Standard_LRS
   ```

2. **Configure Static Website:**
   ```bash
   az storage blob service-properties update --account-name sreyascore --static-website --index-document index.html --404-document 404.html
   ```

## Deployment Automation & CI/CD

### GitHub Actions Workflow
Create `.github/workflows/deploy.yml`:
```yaml
name: Deploy to Production

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Build website
      run: npm run build
    
    - name: Deploy to Netlify
      uses: nwtgck/actions-netlify@v2.0
      with:
        publish-dir: './dist'
        production-branch: main
        github-token: ${{ secrets.GITHUB_TOKEN }}
        deploy-message: "Deploy from GitHub Actions"
      env:
        NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
        NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

### GitLab CI/CD Pipeline
Create `.gitlab-ci.yml`:
```yaml
stages:
  - build
  - deploy

build:
  stage: build
  image: node:18
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour

deploy:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
  script:
    - scp -r dist/* user@your-server:/var/www/sreyascore.net/
  only:
    - main
```

### Automated Deployment Scripts

#### Bash Deployment Script
Create `deploy.sh`:
```bash
#!/bin/bash

# Configuration
REMOTE_HOST="your-server.com"
REMOTE_USER="username"
REMOTE_PATH="/var/www/sreyascore.net"
LOCAL_PATH="./website"

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo -e "${YELLOW}Starting deployment to $REMOTE_HOST...${NC}"

# Check if local files exist
if [ ! -d "$LOCAL_PATH" ]; then
    echo -e "${RED}Error: Local path $LOCAL_PATH does not exist${NC}"
    exit 1
fi

# Create backup
echo -e "${YELLOW}Creating backup...${NC}"
ssh $REMOTE_USER@$REMOTE_HOST "cp -r $REMOTE_PATH ${REMOTE_PATH}_backup_$(date +%Y%m%d_%H%M%S)"

# Deploy files
echo -e "${YELLOW}Deploying files...${NC}"
rsync -avz --delete --exclude='.git' --exclude='node_modules' $LOCAL_PATH/ $REMOTE_USER@$REMOTE_HOST:$REMOTE_PATH/

# Set permissions
echo -e "${YELLOW}Setting permissions...${NC}"
ssh $REMOTE_USER@$REMOTE_HOST "chmod -R 755 $REMOTE_PATH && chmod 644 $REMOTE_PATH/*.html $REMOTE_PATH/*.css $REMOTE_PATH/*.js"

# Test deployment
echo -e "${YELLOW}Testing deployment...${NC}"
if curl -s -f "https://sreyascore.net" > /dev/null; then
    echo -e "${GREEN}Deployment successful!${NC}"
else
    echo -e "${RED}Deployment failed!${NC}"
    exit 1
fi
```

#### PowerShell Deployment Script (Windows)
Create `deploy.ps1`:
```powershell
# Configuration
$RemoteHost = "your-server.com"
$RemoteUser = "username"
$RemotePath = "/var/www/sreyascore.net"
$LocalPath = ".\website"

Write-Host "Starting deployment to $RemoteHost..." -ForegroundColor Yellow

# Check if local files exist
if (-not (Test-Path $LocalPath)) {
    Write-Host "Error: Local path $LocalPath does not exist" -ForegroundColor Red
    exit 1
}

# Create backup
Write-Host "Creating backup..." -ForegroundColor Yellow
ssh $RemoteUser@$RemoteHost "cp -r $RemotePath ${RemotePath}_backup_$(Get-Date -Format 'yyyyMMdd_HHmmss')"

# Deploy files using WinSCP or similar
Write-Host "Deploying files..." -ForegroundColor Yellow
# Note: You'll need to install WinSCP or use alternative method for Windows

Write-Host "Deployment completed!" -ForegroundColor Green
```

## Database Deployment (If Applicable)

### MySQL/MariaDB Setup
```bash
# Install database
sudo apt install mysql-server mariadb-server

# Secure installation
sudo mysql_secure_installation

# Create database and user
sudo mysql -u root -p
CREATE DATABASE sreyascore;
CREATE USER 'sreyascore_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON sreyascore.* TO 'sreyascore_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### PostgreSQL Setup
```bash
# Install PostgreSQL
sudo apt install postgresql postgresql-contrib

# Create database and user
sudo -u postgres psql
CREATE DATABASE sreyascore;
CREATE USER sreyascore_user WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE sreyascore TO sreyascore_user;
\q
```

### Database Migration Scripts
Create `migrations/001_initial_schema.sql`:
```sql
-- Initial database schema
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS downloads (
    id SERIAL PRIMARY KEY,
    filename VARCHAR(255) NOT NULL,
    download_count INTEGER DEFAULT 0,
    last_downloaded TIMESTAMP
);
```

## Enhanced Security Configuration

### Advanced Nginx Security
```nginx
server {
    listen 443 ssl http2;
    server_name sreyascore.net www.sreyascore.net;
    
    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/sreyascore.net/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/sreyascore.net/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    
    # Security Headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # Rate Limiting
    limit_req_zone $binary_remote_addr zone=login:10m rate=10r/m;
    limit_req_zone $binary_remote_addr zone=download:10m rate=30r/m;
    
    location /login {
        limit_req zone=login burst=20 nodelay;
        # ... other config
    }
    
    location /downloads/ {
        limit_req zone=download burst=50 nodelay;
        # ... other config
    }
    
    # Block bad bots
    if ($http_user_agent ~* (bot|crawler|spider|scraper)) {
        return 403;
    }
}
```

### Apache Security (.htaccess)
```apache
# Security Headers
<IfModule mod_headers.c>
    Header always set X-Content-Type-Options nosniff
    Header always set X-Frame-Options DENY
    Header always set X-XSS-Protection "1; mode=block"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
</IfModule>

# Block access to sensitive files
<FilesMatch "\.(htaccess|htpasswd|ini|log|sh|inc|bak)$">
    Order Allow,Deny
    Deny from all
</FilesMatch>

# Prevent directory browsing
Options -Indexes

# Block bad user agents
RewriteEngine On
RewriteCond %{HTTP_USER_AGENT} ^$ [OR]
RewriteCond %{HTTP_USER_AGENT} ^(java|curl|wget) [NC,OR]
RewriteCond %{HTTP_USER_AGENT} ^.*(libwww-perl|curl|wget|python|nikto|wkito|pikto|acunetix).* [NC,OR]
RewriteCond %{HTTP_USER_AGENT} ^.*(winhttp|HTTrack|clshttp|archiver|loader|email|harvest|extract|grab|miner).* [NC]
RewriteRule .* - [F,L]
```

## Advanced Performance Optimization

### Image Optimization
```bash
# Install optimization tools
sudo apt install jpegoptim optipng webp

# Optimize images
find . -name "*.jpg" -exec jpegoptim --strip-all {} \;
find . -name "*.png" -exec optipng -o5 {} \;
find . -name "*.jpg" -exec cwebp -q 80 {} -o {}.webp \;
```

### Critical CSS Inlining
Create `build/critical-css.js`:
```javascript
const critical = require('critical');
const fs = require('fs');

critical.generate({
    src: 'index.html',
    target: 'index-critical.html',
    inline: true,
    dimensions: [
        { width: 320, height: 568 },
        { width: 768, height: 1024 },
        { width: 1920, height: 1080 }
    ]
});
```

### Service Worker for Offline Support
Create `sw.js`:
```javascript
const CACHE_NAME = 'sreyascore-v1';
const urlsToCache = [
    '/',
    '/css/style.css',
    '/js/main.js',
    '/images/logo.png'
];

self.addEventListener('install', event => {
    event.waitUntil(
        caches.open(CACHE_NAME)
            .then(cache => cache.addAll(urlsToCache))
    );
});

self.addEventListener('fetch', event => {
    event.respondWith(
        caches.match(event.request)
            .then(response => response || fetch(event.request))
    );
});
```

## Enhanced Monitoring & Analytics

### Advanced Uptime Monitoring
Create `monitoring/uptime-checker.js`:
```javascript
const https = require('https');
const nodemailer = require('nodemailer');

const checkWebsite = (url) => {
    return new Promise((resolve, reject) => {
        const start = Date.now();
        
        https.get(url, (res) => {
            const responseTime = Date.now() - start;
            const status = res.statusCode;
            
            resolve({
                status,
                responseTime,
                timestamp: new Date().toISOString()
            });
        }).on('error', (err) => {
            reject(err);
        });
    });
};

const sendAlert = async (message) => {
    // Configure your email service
    const transporter = nodemailer.createTransporter({
        service: 'gmail',
        auth: {
            user: 'your-email@gmail.com',
            pass: 'your-app-password'
        }
    });
    
    await transporter.sendMail({
        from: 'alerts@sreyascore.net',
        to: 'admin@sreyascore.net',
        subject: 'Website Alert',
        text: message
    });
};

// Check every 5 minutes
setInterval(async () => {
    try {
        const result = await checkWebsite('https://sreyascore.net');
        
        if (result.status !== 200) {
            await sendAlert(`Website returned status ${result.status}`);
        }
        
        if (result.responseTime > 5000) {
            await sendAlert(`Website response time: ${result.responseTime}ms`);
        }
        
        console.log(`Check: ${result.status} - ${result.responseTime}ms`);
    } catch (error) {
        await sendAlert(`Website check failed: ${error.message}`);
    }
}, 5 * 60 * 1000);
```

### Performance Monitoring Dashboard
Create `monitoring/dashboard.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SreyasCore Monitoring Dashboard</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .metric { display: inline-block; margin: 10px; padding: 20px; border: 1px solid #ccc; }
        .chart-container { width: 600px; height: 400px; margin: 20px 0; }
    </style>
</head>
<body>
    <h1>SreyasCore Website Monitoring</h1>
    
    <div class="metric">
        <h3>Response Time</h3>
        <div id="responseTime">--</div>
    </div>
    
    <div class="metric">
        <h3>Uptime</h3>
        <div id="uptime">--</div>
    </div>
    
    <div class="chart-container">
        <canvas id="performanceChart"></canvas>
    </div>
    
    <script>
        // Performance monitoring code
        const ctx = document.getElementById('performanceChart').getContext('2d');
        const chart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: [],
                datasets: [{
                    label: 'Response Time (ms)',
                    data: [],
                    borderColor: 'rgb(75, 192, 192)',
                    tension: 0.1
                }]
            },
            options: {
                responsive: true,
                scales: {
                    y: {
                        beginAtZero: true
                    }
                }
            }
        });
        
        // Update metrics every 30 seconds
        setInterval(() => {
            const start = performance.now();
            fetch('/')
                .then(response => {
                    const responseTime = performance.now() - start;
                    document.getElementById('responseTime').textContent = `${responseTime.toFixed(2)}ms`;
                    
                    // Update chart
                    const now = new Date().toLocaleTimeString();
                    chart.data.labels.push(now);
                    chart.data.datasets[0].data.push(responseTime);
                    
                    if (chart.data.labels.length > 20) {
                        chart.data.labels.shift();
                        chart.data.datasets[0].data.shift();
                    }
                    
                    chart.update();
                });
        }, 30000);
    </script>
</body>
</html>
```

## Advanced Troubleshooting

### Comprehensive Error Logging
Create `logs/error-logger.js`:
```javascript
const fs = require('fs');
const path = require('path');

class ErrorLogger {
    constructor() {
        this.logFile = path.join(__dirname, 'errors.log');
        this.maxLogSize = 10 * 1024 * 1024; // 10MB
    }
    
    log(error, context = {}) {
        const logEntry = {
            timestamp: new Date().toISOString(),
            error: error.message,
            stack: error.stack,
            context,
            userAgent: context.userAgent || 'Unknown',
            ip: context.ip || 'Unknown',
            url: context.url || 'Unknown'
        };
        
        const logLine = JSON.stringify(logEntry) + '\n';
        
        // Check log file size
        if (fs.existsSync(this.logFile)) {
            const stats = fs.statSync(this.logFile);
            if (stats.size > this.maxLogSize) {
                // Rotate log file
                const backupFile = this.logFile + '.' + new Date().toISOString().split('T')[0];
                fs.renameSync(this.logFile, backupFile);
            }
        }
        
        fs.appendFileSync(this.logFile, logLine);
        
        // Also log to console in development
        if (process.env.NODE_ENV === 'development') {
            console.error('Error logged:', logEntry);
        }
    }
    
    getRecentErrors(limit = 100) {
        if (!fs.existsSync(this.logFile)) return [];
        
        const content = fs.readFileSync(this.logFile, 'utf8');
        const lines = content.trim().split('\n');
        
        return lines
            .slice(-limit)
            .map(line => {
                try {
                    return JSON.parse(line);
                } catch {
                    return null;
                }
            })
            .filter(Boolean);
    }
}

module.exports = new ErrorLogger();
```

### Health Check Endpoint
Create `health-check.js`:
```javascript
const express = require('express');
const router = express.Router();

router.get('/health', async (req, res) => {
    const health = {
        status: 'OK',
        timestamp: new Date().toISOString(),
        uptime: process.uptime(),
        memory: process.memoryUsage(),
        checks: {}
    };
    
    // Check database connection
    try {
        // Add your database health check here
        health.checks.database = 'OK';
    } catch (error) {
        health.checks.database = 'ERROR';
        health.status = 'DEGRADED';
    }
    
    // Check external services
    try {
        // Add external service checks here
        health.checks.externalServices = 'OK';
    } catch (error) {
        health.checks.externalServices = 'ERROR';
        health.status = 'DEGRADED';
    }
    
    const statusCode = health.status === 'OK' ? 200 : 503;
    res.status(statusCode).json(health);
});

module.exports = router;
```

## Cost Optimization Strategies

### Multi-Cloud Cost Analysis
```bash
# AWS Cost Calculator
# S3 Standard: $0.023 per GB/month
# CloudFront: $0.085 per GB (first 10TB)
# Route 53: $0.50 per hosted zone/month

# Google Cloud Cost Calculator
# Cloud Storage: $0.020 per GB/month
# Cloud CDN: $0.075 per GB (first 10TB)
# Cloud DNS: $0.40 per managed zone/month

# Azure Cost Calculator
# Blob Storage: $0.0184 per GB/month
# CDN: $0.081 per GB (first 10TB)
# DNS: $0.50 per hosted zone/month
```

### Resource Scaling Scripts
Create `scripts/scale-resources.js`:
```javascript
const AWS = require('aws-sdk');
const cloudwatch = new AWS.CloudWatch();

const scaleResources = async () => {
    try {
        // Get current metrics
        const metrics = await cloudwatch.getMetricStatistics({
            Namespace: 'AWS/S3',
            MetricName: 'NumberOfObjects',
            Dimensions: [{ Name: 'BucketName', Value: 'sreyascore.net' }],
            StartTime: new Date(Date.now() - 3600000), // 1 hour ago
            EndTime: new Date(),
            Period: 300, // 5 minutes
            Statistics: ['Average']
        }).promise();
        
        // Scale based on metrics
        if (metrics.Datapoints && metrics.Datapoints.length > 0) {
            const avgObjects = metrics.Datapoints[0].Average;
            
            if (avgObjects > 10000) {
                console.log('High traffic detected, scaling up...');
                // Add scaling logic here
            } else if (avgObjects < 1000) {
                console.log('Low traffic, scaling down...');
                // Add scaling logic here
            }
        }
    } catch (error) {
        console.error('Scaling error:', error);
    }
};

// Run every 15 minutes
setInterval(scaleResources, 15 * 60 * 1000);
```

## Containerization & Docker Deployment

### Dockerfile Configuration
Create `Dockerfile`:
```dockerfile
# Multi-stage build for optimization
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine

# Copy built files
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy custom nginx config
COPY nginx.conf /etc/nginx/nginx.conf

# Security: run as non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
USER nextjs

EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose for Development
Create `docker-compose.yml`:
```yaml
version: '3.8'

services:
  website:
    build: .
    ports:
      - "8080:8080"
    environment:
      - NODE_ENV=development
    volumes:
      - ./website:/usr/share/nginx/html
      - ./nginx.conf:/etc/nginx/nginx.conf
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
      - ./logs:/var/log/nginx
    depends_on:
      - website
    restart: unless-stopped

  monitoring:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    restart: unless-stopped

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-storage:/var/lib/grafana
    restart: unless-stopped

volumes:
  grafana-storage:
```

### Docker Deployment Scripts
Create `scripts/docker-deploy.sh`:
```bash
#!/bin/bash

# Docker deployment script
set -e

echo "🚀 Starting Docker deployment..."

# Build and tag images
docker build -t sreyascore:latest .
docker tag sreyascore:latest sreyascore:$(date +%Y%m%d_%H%M%S)

# Stop existing containers
docker-compose down

# Remove old images (keep last 5)
docker images sreyascore --format "table {{.Tag}}" | tail -n +6 | xargs -I {} docker rmi sreyascore:{}

# Start new deployment
docker-compose up -d

# Health check
echo "🔍 Performing health check..."
sleep 10

if curl -f http://localhost:8080/health > /dev/null 2>&1; then
    echo "✅ Deployment successful!"
else
    echo "❌ Deployment failed!"
    docker-compose logs website
    exit 1
fi

echo "🎉 Docker deployment completed!"
```

## Kubernetes Orchestration

### Kubernetes Manifests
Create `k8s/deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sreyascore-website
  labels:
    app: sreyascore
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sreyascore
  template:
    metadata:
      labels:
        app: sreyascore
        version: v1
    spec:
      containers:
      - name: website
        image: sreyascore:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        env:
        - name: NODE_ENV
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: sreyascore-secrets
              key: database-url
```

Create `k8s/service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: sreyascore-service
  labels:
    app: sreyascore
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  selector:
    app: sreyascore
```

Create `k8s/ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sreyascore-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - sreyascore.net
    - www.sreyascore.net
    secretName: sreyascore-tls
  rules:
  - host: sreyascore.net
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: sreyascore-service
            port:
              number: 80
  - host: www.sreyascore.net
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: sreyascore-service
            port:
              number: 80
```

### Kubernetes Deployment Script
Create `scripts/k8s-deploy.sh`:
```bash
#!/bin/bash

# Kubernetes deployment script
set -e

NAMESPACE="sreyascore"
DEPLOYMENT_NAME="sreyascore-website"

echo "🚀 Starting Kubernetes deployment..."

# Create namespace if it doesn't exist
kubectl create namespace $NAMESPACE --dry-run=client -o yaml | kubectl apply -f -

# Apply secrets
kubectl apply -f k8s/secrets.yaml -n $NAMESPACE

# Apply configurations
kubectl apply -f k8s/configmap.yaml -n $NAMESPACE

# Deploy application
kubectl apply -f k8s/deployment.yaml -n $NAMESPACE
kubectl apply -f k8s/service.yaml -n $NAMESPACE
kubectl apply -f k8s/ingress.yaml -n $NAMESPACE

# Wait for deployment to be ready
echo "⏳ Waiting for deployment to be ready..."
kubectl rollout status deployment/$DEPLOYMENT_NAME -n $NAMESPACE

# Scale up gradually for zero-downtime
echo "📈 Scaling up deployment..."
kubectl scale deployment $DEPLOYMENT_NAME --replicas=5 -n $NAMESPACE

# Wait for new pods to be ready
sleep 30

# Scale down old deployment
kubectl scale deployment $DEPLOYMENT_NAME --replicas=3 -n $NAMESPACE

echo "✅ Kubernetes deployment completed!"

# Show deployment status
kubectl get pods -n $NAMESPACE
kubectl get services -n $NAMESPACE
kubectl get ingress -n $NAMESPACE
```

## Multi-Environment Deployment

### Environment Configuration
Create `config/environments/`:

**Development (`config/environments/development.json`):**
```json
{
  "environment": "development",
  "apiUrl": "http://localhost:3000",
  "database": {
    "host": "localhost",
    "port": 5432,
    "name": "sreyascore_dev"
  },
  "features": {
    "debug": true,
    "analytics": false,
    "monitoring": false
  },
  "security": {
    "ssl": false,
    "rateLimit": 1000
  }
}
```

**Staging (`config/environments/staging.json`):**
```json
{
  "environment": "staging",
  "apiUrl": "https://staging-api.sreyascore.net",
  "database": {
    "host": "staging-db.sreyascore.net",
    "port": 5432,
    "name": "sreyascore_staging"
  },
  "features": {
    "debug": false,
    "analytics": true,
    "monitoring": true
  },
  "security": {
    "ssl": true,
    "rateLimit": 500
  }
}
```

**Production (`config/environments/production.json`):**
```json
{
  "environment": "production",
  "apiUrl": "https://api.sreyascore.net",
  "database": {
    "host": "production-db.sreyascore.net",
    "port": 5432,
    "name": "sreyascore_production"
  },
  "features": {
    "debug": false,
    "analytics": true,
    "monitoring": true
  },
  "security": {
    "ssl": true,
    "rateLimit": 100
  }
}
```

### Environment Deployment Script
Create `scripts/deploy-env.sh`:
```bash
#!/bin/bash

# Multi-environment deployment script
set -e

ENVIRONMENT=${1:-development}
CONFIG_FILE="config/environments/${ENVIRONMENT}.json"

if [ ! -f "$CONFIG_FILE" ]; then
    echo "❌ Environment config not found: $CONFIG_FILE"
    exit 1
fi

echo "🚀 Deploying to $ENVIRONMENT environment..."

# Load environment configuration
export $(cat $CONFIG_FILE | jq -r 'to_entries | .[] | .key + "=" + .value')

# Validate configuration
if [ -z "$environment" ]; then
    echo "❌ Invalid environment configuration"
    exit 1
fi

# Environment-specific deployment steps
case $ENVIRONMENT in
    "development")
        echo "🔧 Deploying to development..."
        docker-compose -f docker-compose.dev.yml up -d
        ;;
    "staging")
        echo "🧪 Deploying to staging..."
        kubectl apply -f k8s/staging/ -n staging
        ;;
    "production")
        echo "🌐 Deploying to production..."
        kubectl apply -f k8s/production/ -n production
        
        # Production-specific steps
        kubectl rollout restart deployment/sreyascore-website -n production
        kubectl rollout status deployment/sreyascore-website -n production
        
        # Verify deployment
        ./scripts/health-check.sh production
        ;;
    *)
        echo "❌ Unknown environment: $ENVIRONMENT"
        exit 1
        ;;
esac

echo "✅ $ENVIRONMENT deployment completed!"
```

## Advanced Backup & Disaster Recovery

### Automated Backup System
Create `scripts/backup-system.sh`:
```bash
#!/bin/bash

# Comprehensive backup system
set -e

BACKUP_DIR="/backups/sreyascore"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

echo "💾 Starting backup process..."

# Create backup directory
mkdir -p $BACKUP_DIR

# Database backup (if applicable)
if command -v pg_dump &> /dev/null; then
    echo "🗄️ Backing up database..."
    pg_dump -h localhost -U sreyascore_user sreyascore > $BACKUP_DIR/database_$DATE.sql
    gzip $BACKUP_DIR/database_$DATE.sql
fi

# Website files backup
echo "📁 Backing up website files..."
tar -czf $BACKUP_DIR/website_$DATE.tar.gz -C /var/www sreyascore.net/

# Configuration backup
echo "⚙️ Backing up configurations..."
tar -czf $BACKUP_DIR/config_$DATE.tar.gz \
    /etc/nginx/sites-available/sreyascore.net \
    /etc/ssl/certs/sreyascore.net* \
    /etc/letsencrypt/live/sreyascore.net/

# Logs backup
echo "📝 Backing up logs..."
tar -czf $BACKUP_DIR/logs_$DATE.tar.gz /var/log/nginx/ /var/log/letsencrypt/

# Create backup manifest
cat > $BACKUP_DIR/manifest_$DATE.txt << EOF
Backup created: $(date)
Environment: Production
Components:
- Database: $(ls -la $BACKUP_DIR/database_$DATE.sql.gz)
- Website: $(ls -la $BACKUP_DIR/website_$DATE.tar.gz)
- Config: $(ls -la $BACKUP_DIR/config_$DATE.tar.gz)
- Logs: $(ls -la $BACKUP_DIR/logs_$DATE.tar.gz)
EOF

# Cleanup old backups
echo "🧹 Cleaning up old backups..."
find $BACKUP_DIR -name "*.gz" -mtime +$RETENTION_DAYS -delete
find $BACKUP_DIR -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete

# Verify backup integrity
echo "🔍 Verifying backup integrity..."
for file in $BACKUP_DIR/*_$DATE.*; do
    if [[ $file == *.gz ]]; then
        gunzip -t "$file" && echo "✅ $file integrity verified" || echo "❌ $file integrity check failed"
    elif [[ $file == *.tar.gz ]]; then
        tar -tzf "$file" > /dev/null && echo "✅ $file integrity verified" || echo "❌ $file integrity check failed"
    fi
done

echo "✅ Backup completed successfully!"
echo "📊 Backup size: $(du -sh $BACKUP_DIR | cut -f1)"
echo "🗂️ Backup location: $BACKUP_DIR"
```

### Disaster Recovery Plan
Create `docs/disaster-recovery-plan.md`:
```markdown
# SreyasCore Disaster Recovery Plan

## Recovery Time Objectives (RTO)
- **Critical Systems**: 1 hour
- **Website**: 4 hours
- **Full Infrastructure**: 24 hours

## Recovery Point Objectives (RPO)
- **Database**: 15 minutes
- **Website Files**: 1 hour
- **Configuration**: 4 hours

## Recovery Procedures

### 1. Website Down Scenario
1. **Immediate Response (0-15 minutes)**
   - Check monitoring alerts
   - Verify server status
   - Contact hosting provider if needed

2. **Quick Recovery (15-60 minutes)**
   - Restore from latest backup
   - Verify functionality
   - Update status page

3. **Full Recovery (1-4 hours)**
   - Investigate root cause
   - Implement permanent fix
   - Update documentation

### 2. Database Corruption
1. **Stop affected services**
2. **Restore from latest backup**
3. **Verify data integrity**
4. **Resume services**

### 3. Complete Infrastructure Failure
1. **Activate backup infrastructure**
2. **Restore from offsite backups**
3. **Update DNS records**
4. **Verify all services**

## Contact Information
- **Primary Admin**: [Contact Info]
- **Backup Admin**: [Contact Info]
- **Hosting Provider**: [Contact Info]
- **Domain Provider**: [Contact Info]

## Recovery Checklist
- [ ] Services stopped
- [ ] Root cause identified
- [ ] Backup integrity verified
- [ ] Recovery procedure executed
- [ ] Services tested
- [ ] Monitoring restored
- [ ] Documentation updated
- [ ] Post-mortem scheduled
```

## Compliance & Audit Considerations

### Security Compliance Checklist
Create `docs/compliance-checklist.md`:
```markdown
# Security Compliance Checklist

## GDPR Compliance
- [ ] Data processing register maintained
- [ ] Privacy policy updated
- [ ] Cookie consent implemented
- [ ] Data subject rights procedures
- [ ] Data breach notification plan
- [ ] Data retention policies

## PCI DSS (If handling payments)
- [ ] Cardholder data encrypted
- [ ] Access controls implemented
- [ ] Security monitoring active
- [ ] Vulnerability management
- [ ] Incident response plan
- [ ] Security policies documented

## SOC 2 Type II
- [ ] Security controls documented
- [ ] Access management procedures
- [ ] Change management process
- [ ] Risk assessment completed
- [ ] Vendor management program
- [ ] Business continuity plan

## ISO 27001
- [ ] Information security policy
- [ ] Risk assessment framework
- [ ] Security controls implemented
- [ ] Monitoring and review process
- [ ] Continuous improvement plan
```

### Audit Trail System
Create `scripts/audit-logger.js`:
```javascript
const fs = require('fs');
const crypto = require('crypto');

class AuditLogger {
    constructor() {
        this.logFile = '/var/log/sreyascore/audit.log';
        this.ensureLogDirectory();
    }
    
    ensureLogDirectory() {
        const dir = require('path').dirname(this.logFile);
        if (!fs.existsSync(dir)) {
            fs.mkdirSync(dir, { recursive: true });
        }
    }
    
    log(action, user, details, ip) {
        const auditEntry = {
            timestamp: new Date().toISOString(),
            action: action,
            user: user || 'system',
            ip: ip || 'unknown',
            details: details,
            sessionId: this.generateSessionId(),
            hash: this.generateHash(action, user, details)
        };
        
        const logLine = JSON.stringify(auditEntry) + '\n';
        fs.appendFileSync(this.logFile, logLine);
        
        // Also log to syslog for system-level auditing
        require('syslog').log('info', `AUDIT: ${action} by ${user} from ${ip}`);
    }
    
    generateSessionId() {
        return crypto.randomBytes(16).toString('hex');
    }
    
    generateHash(action, user, details) {
        const data = `${action}${user}${JSON.stringify(details)}${new Date().toISOString()}`;
        return crypto.createHash('sha256').update(data).digest('hex');
    }
    
    searchAuditLog(criteria) {
        if (!fs.existsSync(this.logFile)) return [];
        
        const content = fs.readFileSync(this.logFile, 'utf8');
        const lines = content.trim().split('\n');
        
        return lines
            .map(line => {
                try {
                    return JSON.parse(line);
                } catch {
                    return null;
                }
            })
            .filter(Boolean)
            .filter(entry => {
                if (criteria.action && entry.action !== criteria.action) return false;
                if (criteria.user && entry.user !== criteria.user) return false;
                if (criteria.ip && entry.ip !== criteria.ip) return false;
                if (criteria.startDate && new Date(entry.timestamp) < new Date(criteria.startDate)) return false;
                if (criteria.endDate && new Date(entry.timestamp) > new Date(criteria.endDate)) return false;
                return true;
            });
    }
    
    generateAuditReport(startDate, endDate) {
        const entries = this.searchAuditLog({ startDate, endDate });
        
        const report = {
            period: { startDate, endDate },
            totalActions: entries.length,
            actionsByUser: {},
            actionsByType: {},
            topIPs: {},
            summary: {}
        };
        
        entries.forEach(entry => {
            // Count by user
            report.actionsByUser[entry.user] = (report.actionsByUser[entry.user] || 0) + 1;
            
            // Count by action type
            report.actionsByType[entry.action] = (report.actionsByType[entry.action] || 0) + 1;
            
            // Count by IP
            report.topIPs[entry.ip] = (report.topIPs[entry.ip] || 0) + 1;
        });
        
        return report;
    }
}

module.exports = new AuditLogger();
```

### Compliance Monitoring Dashboard
Create `monitoring/compliance-dashboard.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SreyasCore Compliance Dashboard</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; background: #f5f5f5; }
        .container { max-width: 1200px; margin: 0 auto; }
        .header { background: #fff; padding: 20px; border-radius: 8px; margin-bottom: 20px; }
        .metrics { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 20px; margin-bottom: 20px; }
        .metric { background: #fff; padding: 20px; border-radius: 8px; text-align: center; }
        .metric h3 { margin: 0 0 10px 0; color: #333; }
        .metric .value { font-size: 2em; font-weight: bold; color: #007bff; }
        .chart-container { background: #fff; padding: 20px; border-radius: 8px; margin-bottom: 20px; }
        .compliance-status { background: #fff; padding: 20px; border-radius: 8px; }
        .status-item { display: flex; justify-content: space-between; align-items: center; padding: 10px 0; border-bottom: 1px solid #eee; }
        .status-pass { color: #28a745; }
        .status-fail { color: #dc3545; }
        .status-warning { color: #ffc107; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🔒 SreyasCore Compliance Dashboard</h1>
            <p>Real-time compliance monitoring and audit trail</p>
        </div>
        
        <div class="metrics">
            <div class="metric">
                <h3>Total Audit Events</h3>
                <div class="value" id="totalEvents">--</div>
            </div>
            <div class="metric">
                <h3>Active Users</h3>
                <div class="value" id="activeUsers">--</div>
            </div>
            <div class="metric">
                <h3>Security Score</h3>
                <div class="value" id="securityScore">--</div>
            </div>
            <div class="metric">
                <h3>Compliance Status</h3>
                <div class="value" id="complianceStatus">--</div>
            </div>
        </div>
        
        <div class="chart-container">
            <h3>Audit Events Over Time</h3>
            <canvas id="auditChart" height="100"></canvas>
        </div>
        
        <div class="compliance-status">
            <h3>Compliance Checklist</h3>
            <div class="status-item">
                <span>GDPR Compliance</span>
                <span class="status-pass">✅ PASS</span>
            </div>
            <div class="status-item">
                <span>SSL/TLS Configuration</span>
                <span class="status-pass">✅ PASS</span>
            </div>
            <div class="status-item">
                <span>Security Headers</span>
                <span class="status-pass">✅ PASS</span>
            </div>
            <div class="status-item">
                <span>Access Controls</span>
                <span class="status-pass">✅ PASS</span>
            </div>
            <div class="status-item">
                <span>Backup Procedures</span>
                <span class="status-pass">✅ PASS</span>
            </div>
            <div class="status-item">
                <span>Incident Response Plan</span>
                <span class="status-warning">⚠️ IN PROGRESS</span>
            </div>
        </div>
    </div>
    
    <script>
        // Compliance monitoring code
        const ctx = document.getElementById('auditChart').getContext('2d');
        const chart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: [],
                datasets: [{
                    label: 'Audit Events',
                    data: [],
                    borderColor: 'rgb(75, 192, 192)',
                    backgroundColor: 'rgba(75, 192, 192, 0.1)',
                    tension: 0.1
                }]
            },
            options: {
                responsive: true,
                scales: {
                    y: {
                        beginAtZero: true
                    }
                }
            }
        });
        
        // Update metrics every 30 seconds
        setInterval(() => {
            // Simulate data updates
            const now = new Date().toLocaleTimeString();
            chart.data.labels.push(now);
            chart.data.datasets[0].data.push(Math.floor(Math.random() * 50) + 10);
            
            if (chart.data.labels.length > 20) {
                chart.data.labels.shift();
                chart.data.datasets[0].data.shift();
            }
            
            chart.update();
            
            // Update metrics
            document.getElementById('totalEvents').textContent = Math.floor(Math.random() * 1000) + 500;
            document.getElementById('activeUsers').textContent = Math.floor(Math.random() * 50) + 10;
            document.getElementById('securityScore').textContent = Math.floor(Math.random() * 20) + 80;
        }, 30000);
        
        // Initial load
        document.getElementById('totalEvents').textContent = '847';
        document.getElementById('activeUsers').textContent = '23';
        document.getElementById('securityScore').textContent = '92';
        document.getElementById('complianceStatus').textContent = '95%';
    </script>
</body>
</html>
```

## Final Enhanced Deployment Checklist

### Pre-Deployment Testing (Enhanced)
- [ ] **Local Testing:**
  - [ ] Website loads correctly on localhost
  - [ ] All links and navigation working
  - [ ] Contact forms functional
  - [ ] Download links working
  - [ ] Mobile responsiveness tested
  - [ ] Cross-browser compatibility verified
  - [ ] Docker containers build successfully
  - [ ] Kubernetes manifests validated

- [ ] **Performance Testing:**
  - [ ] Page load times under 3 seconds
  - [ ] Images optimized and compressed
  - [ ] CSS and JavaScript minified
  - [ ] Gzip compression enabled
  - [ ] Browser caching configured
  - [ ] CDN integration tested
  - [ ] Service worker functionality verified

- [ ] **Security Testing:**
  - [ ] SSL certificate valid
  - [ ] Security headers configured
  - [ ] File permissions set correctly
  - [ ] Sensitive files protected
  - [ ] SQL injection protection (if applicable)
  - [ ] Rate limiting tested
  - [ ] Bot protection verified
  - [ ] Compliance requirements met

### Deployment Verification (Enhanced)
- [ ] **DNS Configuration:**
  - [ ] Domain points to correct server
  - [ ] www subdomain configured
  - [ ] DNS propagation completed
  - [ ] SSL certificate working
  - [ ] CDN configuration verified

- [ ] **Website Functionality:**
  - [ ] Homepage loads correctly
  - [ ] All pages accessible
  - [ ] Forms submit successfully
  - [ ] Downloads work properly
  - [ ] Search functionality (if applicable)
  - [ ] API endpoints functional (if applicable)
  - [ ] Database connections working

- [ ] **Performance Verification:**
  - [ ] Google PageSpeed Insights score > 90
  - [ ] GTmetrix grade A
  - [ ] Mobile-friendly test passed
  - [ ] Core Web Vitals optimized
  - [ ] CDN performance verified
  - [ ] Load testing completed

### Post-Deployment Monitoring (Enhanced)
- [ ] **Uptime Monitoring:**
  - [ ] Uptime monitoring service configured
  - [ ] Alert notifications set up
  - [ ] Response time monitoring active
  - [ ] Error tracking implemented
  - [ ] Performance monitoring active
  - [ ] Security monitoring configured

- [ ] **Analytics & SEO:**
  - [ ] Google Analytics installed
  - [ ] Google Search Console configured
  - [ ] Sitemap submitted
  - [ ] Robots.txt configured
  - [ ] Meta tags optimized
  - [ ] Structured data implemented

- [ ] **Backup & Recovery:**
  - [ ] Automated backup system configured
  - [ ] Backup restoration tested
  - [ ] Disaster recovery plan documented
  - [ ] Contact information updated
  - [ ] Recovery procedures tested
  - [ ] Offsite backup verified

- [ ] **Compliance & Audit:**
  - [ ] Audit logging active
  - [ ] Compliance monitoring configured
  - [ ] Security policies implemented
  - [ ] Regular security scans scheduled
  - [ ] Compliance reports generated

---

**Need help with deployment?** Contact our support team at support@sreyascore.net or join our Discord community for assistance.

**Last Updated:** December 2024  
**Version:** 3.0  
**Maintainer:** SreyasCore Development Team 