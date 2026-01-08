## 1. use validation middleware per route 

```javascript
import { body, param, validationResult } from 'express-validator';

export function validateRequest(req, res, next) {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      statusCode: 400,
      errors: errors.array().map(err => ({
        field: err.param,
        message: err.msg,
        value: err.value,
      })),
    });
  }
  next();
}
```

## 2. define validation rules per endpoint 

```javascript
router.post(
  '/users',
  [
    body('email')
      .isEmail().withMessage('Invalid email format')
      .normalizeEmail(),
    body('password')
      .isLength({ min: 8 }).withMessage('Password must be at least 8 characters')
      .matches(/[A-Z]/).withMessage('Password must contain uppercase')
      .matches(/[0-9]/).withMessage('Password must contain number')
      .matches(/[!@#$%^&*]/).withMessage('Password must contain special character'),
    body('firstName')
      .trim()
      .notEmpty().withMessage('First name is required')
      .isLength({ min: 2 }).withMessage('First name must be at least 2 characters'),
    body('lastName')
      .trim()
      .notEmpty().withMessage('Last name is required'),
  ],
  validateRequest,
  userController.create
);
```

## 3. try to use schema validation libraries

```javascript
import Joi from 'joi';

export const createUserSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).pattern(/[A-Z]/).pattern(/[0-9]/).required(),
  firstName: Joi.string().min(2).required(),
  lastName: Joi.string().min(2).required(),
});
```

## 4. centralize the error handling middleware

```javascript
export function errorHandler(err, req, res, next) {
  const requestId = res.getHeader('x-request-id');
  
  // Determine status code and message
  let statusCode = err.statusCode || 500;
  let errorCode = err.errorCode || 'INTERNAL_SERVER_ERROR';
  let message = err.message || 'Internal server error';
  
  // Log the error
  logger.error('Request error', {
    requestId,
    statusCode,
    errorCode,
    message,
    stack: err.stack,
    url: req.url,
    method: req.method,
  });
  
  // Send response
  res.status(statusCode).json({
    success: false,
    statusCode,
    errorCode,
    message,
    requestId,
    timestamp: new Date(),
    // In development only
    ...(process.env.NODE_ENV === 'development' && { details: err.stack }),
  });
}
```

## 5. make sure when you register middleware register error handler after all routes

## 6. write custom error handler classes 

```javascript
export class ValidationError extends Error {
  constructor(message, details = []) {
    super(message);
    this.statusCode = 400;
    this.errorCode = 'VALIDATION_ERROR';
    this.details = details;
  }
}

export class NotFoundError extends Error {
  constructor(resource) {
    super(`${resource} not found`);
    this.statusCode = 404;
    this.errorCode = 'NOT_FOUND';
  }
}

router.get('/users/:id',
  asyncHandler(async (req, res) => {
    const user = await User.findById(req.params.id);
    if (!user) throw new NotFoundError('User');
    res.json(user);
  })
);
```
## 7. use request timeout always

```javascript
app.use((req, res, next) => {
  req.setTimeout(30000);  // 30 seconds
  res.setTimeout(30000);
  next();
});
```

## 8. Implement Circuit breakers
```javascript
import CircuitBreaker from 'opossum';

const externalApiBreaker = new CircuitBreaker(async (endpoint, data) => {
  return axios.post(endpoint, data, { timeout: 5000 });
}, {
  timeout: 5000,
  errorThresholdPercentage: 50,
  resetTimeout: 30000,
  name: 'external-api',
  fallback: () => ({ data: null, cached: true }),
});
```

## 9. Use timeout for database queries
```javascript
async function getUserWithTimeout(userId) {
  return Promise.race([
    User.findById(userId),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Database query timeout')), 10000)
    ),
  ]);
}
```

## 10. Implement request queue to prevent overload 

```javascript
import pLimit from 'p-limit';

const requestLimit = pLimit(100);  // Max 100 concurrent requests

app.use((req, res, next) => {
  requestLimit(() => {
    next();
  }).catch(() => {
    res.status(503).json({ error: 'Service temporarily unavailable' });
  });
});
```

## 11. use consistent response format
```javascript
export function successResponse(data, message = 'Success', statusCode = 200) {
  return {
    success: true,
    statusCode,
    message,
    data,
    timestamp: new Date(),
  };
}

export function errorResponse(message, errorCode = 'ERROR', statusCode = 500, details = null) {
  return {
    success: false,
    statusCode,
    message,
    errorCode,
    details,
    timestamp: new Date(),
  };
}

```

## 12. use pagination metadata for list endpoints

```javascript
export function listResponse(items, page, limit, total) {
  return {
    success: true,
    statusCode: 200,
    data: items,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit),
    },
    timestamp: new Date(),
  };
}
```

## 13. implement api level rate limiting (depends on the requirement), individual requests level also we can configure, it all depends on the business and its usecases.












