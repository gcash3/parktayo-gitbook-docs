# 27 Production

## 4.2 Production Deployment

### Backend Deployment (AWS EC2)

#### 1. Launch EC2 Instance
```bash
# AWS Console:
# - AMI: Ubuntu Server 20.04 LTS
# - Instance Type: t3.medium (2 vCPU, 4 GB RAM) minimum
# - Storage: 30 GB SSD
# - Security Group:
#   - Port 22 (SSH)
#   - Port 80 (HTTP)
#   - Port 443 (HTTPS)
#   - Port 5000 (API) - temporary, will use reverse proxy
```

#### 2. Connect to Instance
```bash
ssh -i your-key.pem ubuntu@your-ec2-ip
```

#### 3. Install Dependencies
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Install PM2
sudo npm install -g pm2

# Install Git
sudo apt install -y git

# Install NGINX
sudo apt install -y nginx
```

#### 4. Clone Repository
```bash
# Create app directory
sudo mkdir -p /var/www/parktayo
sudo chown -R $USER:$USER /var/www/parktayo

# Clone repo
cd /var/www/parktayo
git clone https://github.com/parktayo/parktayo-system.git .
cd parktayo-backend
```

#### 5. Install App Dependencies
```bash
npm install --production
```

#### 6. Configure Environment
```bash
# Create production .env
sudo nano .env
```

**Production .env:**
```bash
PORT=5000
NODE_ENV=production

MONGODB_URI=mongodb+srv://prod_user:prod_pass@prod-cluster.mongodb.net/parktayo_prod

JWT_SECRET=super-secret-production-key-use-random-generator

GOOGLE_MAPS_API_KEY=AIzaSyC_production_key

# Firebase (production)
FIREBASE_SERVICE_ACCOUNT=./config/firebase-prod.json
FIREBASE_PROJECT_ID=parktayo-prod

# Cloudinary
CLOUDINARY_CLOUD_NAME=parktayo-prod
CLOUDINARY_API_KEY=prod_key
CLOUDINARY_API_SECRET=prod_secret

# SMS
SEMAPHORE_API_KEY=production_semaphore_key

# CORS
CORS_ORIGIN=https://admin.parktayo.com,https://www.parktayo.com
```

#### 7. Setup PM2
```bash
# Start app with PM2
pm2 start src/server.js --name parktayo-api

# Configure auto-restart on reboot
pm2 startup
pm2 save

# Monitor
pm2 monit
```

#### 8. Configure NGINX Reverse Proxy
```bash
# Create NGINX config
sudo nano /etc/nginx/sites-available/parktayo-api
```

**NGINX Configuration:**
```nginx
server {
    listen 80;
    server_name api.parktayo.com;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
    limit_req zone=api_limit burst=20 nodelay;

    # Max body size (for file uploads)
    client_max_body_size 10M;
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/parktayo-api /etc/nginx/sites-enabled/

# Test config
sudo nginx -t

# Restart NGINX
sudo systemctl restart nginx
```

#### 9. SSL Certificate (Let's Encrypt)
```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d api.parktayo.com

# Auto-renewal (already setup by certbot)
sudo systemctl status certbot.timer
```

#### 10. Firewall Configuration
```bash
# Setup UFW firewall
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable

# Check status
sudo ufw status
```

---

### Database Setup (MongoDB Atlas)

#### 1. Create Production Cluster
```
1. Go to: https://www.mongodb.com/cloud/atlas
2. Create new cluster:
   - Cloud Provider: AWS
   - Region: Asia Pacific (Singapore)
   - Cluster Tier: M10 or higher
   - Cluster Name: parktayo-prod

3. Database Access:
   - Create user: prod_user
   - Password: strong-random-password
   - Built-in Role: readWrite@parktayo_prod

4. Network Access:
   - Add EC2 IP address
   - Or use VPC Peering for better security

5. Get connection string:
   mongodb+srv://prod_user:password@prod-cluster.mongodb.net/parktayo_prod
```

#### 2. Configure Indexes
```javascript
// Connect to database
mongosh "mongodb+srv://prod-cluster.mongodb.net/parktayo_prod" --username prod_user

// Create indexes
db.users.createIndex({ email: 1 }, { unique: true });
db.users.createIndex({ phoneNumber: 1 }, { sparse: true, unique: true });

db.parkingspaces.createIndex({ location: "2dsphere" });
db.parkingspaces.createIndex({ landlordId: 1, status: 1 });

db.bookings.createIndex({ userId: 1, status: 1 });
db.bookings.createIndex({ parkingSpaceId: 1, startTime: 1, endTime: 1 });
db.bookings.createIndex({ status: 1, createdAt: -1 });

db.transactions.createIndex({ userId: 1, createdAt: -1 });
```

#### 3. Backup Strategy
```bash
# Configure automated backups in Atlas
# Settings → Backup → Enable Cloud Backup
# Retention: 7 days daily, 4 weeks weekly

# Manual backup
mongodump --uri="mongodb+srv://prod-user:password@prod-cluster.mongodb.net/parktayo_prod" --out=/backups/$(date +%Y-%m-%d)
```

---

### Mobile App Deployment

#### Android (Google Play Store)

**1. Prepare for Release**
```bash
cd parktayoflutter

# Update version in pubspec.yaml
version: 1.0.0+1
```

**2. Generate Keystore**
```bash
keytool -genkey -v -keystore ~/parktayo-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias parktayo

# Store keystore password securely
```

**3. Configure Signing**
```bash
# Create key.properties
nano android/key.properties
```

```properties
storePassword=your-keystore-password
keyPassword=your-key-password
keyAlias=parktayo
storeFile=/path/to/parktayo-release-key.jks
```

**4. Build Release APK**
```bash
flutter build apk --release

# Or App Bundle (recommended)
flutter build appbundle --release
```

**5. Upload to Play Store**
```
1. Go to: https://play.google.com/console
2. Create app: ParkTayo
3. Fill app details
4. Upload APK/App Bundle: build/app/outputs/bundle/release/app-release.aab
5. Set pricing: Free
6. Submit for review
```

#### iOS (App Store)

**1. Prepare for Release**
```bash
cd parktayoflutter

# Update version
flutter pub get
```

**2. Configure Xcode**
```bash
# Open Xcode
open ios/Runner.xcworkspace

# In Xcode:
# 1. Select Runner target
# 2. General → Identity
#    - Display Name: ParkTayo
#    - Bundle Identifier: com.parktayo.app
#    - Version: 1.0.0
#    - Build: 1

# 3. Signing & Capabilities
#    - Team: Your Apple Developer Team
#    - Signing Certificate: Apple Distribution

# 4. Build Settings
#    - Set bitcode to Yes
```

**3. Build Release**
```bash
flutter build ios --release

# Or build archive in Xcode
# Product → Archive
```

**4. Upload to App Store**
```
1. In Xcode: Window → Organizer
2. Select archive → Distribute App
3. App Store Connect → Upload
4. Go to: https://appstoreconnect.apple.com
5. Select app → Prepare for Submission
6. Fill metadata
7. Submit for review
```

---

### Admin Dashboard Deployment (Vercel/Netlify)

#### Option A: Vercel

**1. Build Production**
```bash
cd newparktayoadmin
npm run build
```

**2. Deploy to Vercel**
```bash
# Install Vercel CLI
npm install -g vercel

# Login
vercel login

# Deploy
vercel --prod

# Configure environment variables in Vercel dashboard
```

#### Option B: Netlify

**1. Build**
```bash
npm run build
```

**2. Deploy**
```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login
netlify login

# Deploy
netlify deploy --prod --dir=build

# Or connect GitHub repo for auto-deploy
```

#### Option C: Self-Hosted (NGINX)

**1. Build**
```bash
npm run build
```

**2. Deploy to Server**
```bash
# Copy build files to server
scp -r build/* user@server:/var/www/admin.parktayo.com/
```

**3. NGINX Configuration**
```nginx
server {
    listen 80;
    server_name admin.parktayo.com;
    root /var/www/admin.parktayo.com;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/css text/javascript application/javascript application/json;

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

**4. SSL**
```bash
sudo certbot --nginx -d admin.parktayo.com
```

---

### Marketing Website Deployment

**1. Prepare Files**
```bash
cd parktayo-website
```

**2. Deploy to Server**
```bash
# Copy files
scp -r * user@server:/var/www/www.parktayo.com/
```

**3. NGINX Configuration**
```nginx
server {
    listen 80;
    server_name www.parktayo.com parktayo.com;
    root /var/www/www.parktayo.com;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

**4. Install PHP**
```bash
sudo apt install php8.0-fpm php8.0-mysql php8.0-curl
```

**5. SSL**
```bash
sudo certbot --nginx -d www.parktayo.com -d parktayo.com
```

---
