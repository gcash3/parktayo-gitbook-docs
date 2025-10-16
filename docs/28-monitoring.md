# 28 Monitoring

## 4.3 Monitoring & Maintenance

### Application Monitoring

#### PM2 Monitoring
```bash
# View all processes
pm2 list

# Monitor in real-time
pm2 monit

# View logs
pm2 logs parktayo-api

# Restart app
pm2 restart parktayo-api

# Reload (zero-downtime)
pm2 reload parktayo-api
```

#### Setup PM2 Plus (Advanced Monitoring)
```bash
# Link PM2 to PM2 Plus
pm2 link <secret_key> <public_key>

# Access dashboard: https://app.pm2.io
```

---

### Error Tracking (Sentry)

**1. Install Sentry**
```bash
npm install @sentry/node
```

**2. Configure in server.js**
```javascript
const Sentry = require('@sentry/node');

Sentry.init({
  dsn: 'your-sentry-dsn',
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0
});

// Error handler
app.use(Sentry.Handlers.errorHandler());
```

---

### Performance Monitoring

**1. New Relic**
```bash
npm install newrelic

# Create newrelic.js config
```

**2. Datadog**
```bash
npm install --save dd-trace

# Initialize at app start
require('dd-trace').init();
```

---

### Database Monitoring

**MongoDB Atlas:**
- View metrics dashboard
- Set up alerts:
  - CPU usage > 80%
  - Connections > 90% of limit
  - Disk space < 20%

---

### Log Management

**Winston + CloudWatch**
```javascript
const winston = require('winston');
const CloudWatchTransport = require('winston-cloudwatch');

const logger = winston.createLogger({
  transports: [
    new winston.transports.Console(),
    new CloudWatchTransport({
      logGroupName: 'parktayo-api',
      logStreamName: 'production',
      awsRegion: 'ap-southeast-1'
    })
  ]
});
```

---

### Backup Strategy

**1. Database Backups**
```bash
# Daily automated backups in MongoDB Atlas
# Retention: 7 daily, 4 weekly, 12 monthly

# Manual backup script
#!/bin/bash
DATE=$(date +%Y-%m-%d)
mongodump --uri="$MONGODB_URI" --out="/backups/$DATE"
tar -czf "/backups/backup-$DATE.tar.gz" "/backups/$DATE"
rm -rf "/backups/$DATE"

# Upload to S3
aws s3 cp "/backups/backup-$DATE.tar.gz" s3://parktayo-backups/
```

**2. Code Backups**
```bash
# Git repository (GitHub)
# Automated via CI/CD

# Additional backup to S3
aws s3 sync /var/www/parktayo s3://parktayo-code-backup/
```

**3. Media Backups**
```bash
# Cloudinary automatic backup
# Configure in Cloudinary settings
```

---

### Security Maintenance

**1. Regular Updates**
```bash
# Check for updates
npm outdated

# Update dependencies
npm update

# Audit vulnerabilities
npm audit
npm audit fix
```

**2. SSL Certificate Renewal**
```bash
# Auto-renewal by certbot
sudo systemctl status certbot.timer

# Manual renewal
sudo certbot renew

# Test renewal
sudo certbot renew --dry-run
```

**3. Security Scanning**
```bash
# Snyk scan
npx snyk test

# OWASP Dependency Check
npm audit
```

---

### Update Deployment Process

**Zero-Downtime Deployment:**
```bash
# 1. Pull latest code
cd /var/www/parktayo/parktayo-backend
git pull origin main

# 2. Install dependencies
npm install --production

# 3. Run database migrations (if any)
node scripts/migrate.js

# 4. Reload PM2 (zero-downtime)
pm2 reload parktayo-api

# 5. Verify
pm2 logs parktayo-api --lines 50
```

---
