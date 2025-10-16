# Authentication & Security

## JWT Authentication

#### Token Generation

```javascript
const generateToken = (userId, role) => {
  return jwt.sign(
    { id: userId, role },
    process.env.JWT_SECRET,
    { expiresIn: process.env.JWT_EXPIRES_IN || '7d' }
  );
};
```

#### Token Verification Middleware

```javascript
const requireAuth = async (req, res, next) => {
  try {
    // 1. Get token from header
    const token = req.headers.authorization?.split(' ')[1];

    if (!token) {
      return res.status(401).json({
        status: 'error',
        message: 'Authentication required'
      });
    }

    // 2. Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    // 3. Check if user exists
    const user = await User.findById(decoded.id);

    if (!user || !user.isActive) {
      return res.status(401).json({
        status: 'error',
        message: 'User not found or inactive'
      });
    }

    // 4. Attach user to request
    req.user = user;
    next();

  } catch (error) {
    return res.status(401).json({
      status: 'error',
      message: 'Invalid or expired token'
    });
  }
};
```

## Password Hashing

```javascript
const bcrypt = require('bcryptjs');

// Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) {
    return next();
  }

  this.password = await bcrypt.hash(this.password, 12);
  next();
});

// Compare password method
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};
```

## Admin Level Authorization

```javascript
const requireSuperAdmin = async (req, res, next) => {
  const admin = await Admin.findById(req.user.id);

  if (!admin || admin.adminLevel !== 'super_admin') {
    return res.status(403).json({
      status: 'error',
      message: 'Super admin access required'
    });
  }

  next();
};

const requirePermission = (permission) => {
  return async (req, res, next) => {
    const admin = await Admin.findById(req.user.id);

    // Super admin bypasses all checks
    if (admin.adminLevel === 'super_admin') {
      return next();
    }

    // Check permission
    if (!admin.permissions.includes(permission)) {
      return res.status(403).json({
        status: 'error',
        message: 'Permission denied'
      });
    }

    next();
  };
};
```

## Security Best Practices

#### Rate Limiting
```javascript
const rateLimit = require('express-rate-limit');

const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 requests per window
  message: 'Too many login attempts, please try again later'
});

app.use('/api/v1/auth/login', authLimiter);
```

#### Input Sanitization
```javascript
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss');

app.use(mongoSanitize()); // Prevent NoSQL injection
```

#### CORS Configuration
```javascript
const cors = require('cors');

app.use(cors({
  origin: [
    'http://localhost:3000', // Admin dashboard
    'https://admin.parktayo.com',
    'https://www.parktayo.com'
  ],
  credentials: true
}));
```

#### Security Headers
```javascript
const helmet = require('helmet');

app.use(helmet()); // Set security headers
```
