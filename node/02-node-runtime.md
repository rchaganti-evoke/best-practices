## 1. Use asychronous programming with Async/Await

```javascript
// ✅ DO: Use async/await for readable asynchronous code
async function getUserWithOrders(userId) {
  try {
    const user = await User.findById(userId);
    if (!user) throw new NotFoundError('User not found');
    
    const orders = await Order.find({ userId });
    return { user, orders };
  } catch (error) {
    logger.error('Failed to fetch user and orders', error);
    throw error;
  }
}
```

## 2. Do the Parallel operations with Promise.all

```javascript
async function fetchUserData(userId) {
  const [user, orders, payments] = await Promise.all([
    User.findById(userId),
    Order.find({ userId }),
    Payment.find({ userId }),
  ]);
  return { user, orders, payments };
}
```

## 3. handle promises in loops correctly

```javascript
async function processUsers(userIds) {
  for (const userId of userIds) {
    await processUser(userId);  // Sequential when needed
  }
}
```

## 4. do parallel processing when appropriate

```javascript
async function processUsersInParallel(userIds) {
  await Promise.all(
    userIds.map(userId => processUser(userId))
  );
}
```

## 5. Implement proper error handling in Event-Driven code

```javascript
class DataProcessor extends EventEmitter {
  process(data) {
    try {
      const result = this.validateData(data);
      this.emit('processed', result);
    } catch (error) {
      logger.error('Processing error', error);
      this.emit('error', error);
    }
  }
}

const processor = new DataProcessor();
processor.on('error', (error) => {
  logger.error('Data processor error:', error);
  // Notify alert system or trigger recovery
});
```

## 6. handle the stream errors appropriately

```javascript
readStream
  .pipe(transformStream)
  .pipe(writeStream)
  .on('error', (error) => {
    logger.error('Stream error:', error);
  });

readStream.on('error', (error) => {
  logger.error('Read stream error:', error);
});
```

## 7. while processing large files do in chunks

```javascript
async function* readLargeFile(filePath) {
  const stream = fs.createReadStream(filePath, { encoding: 'utf8' });
  let data = '';
  
  for await (const chunk of stream) {
    data += chunk;
    const lines = data.split('\n');
    data = lines.pop();  // Keep incomplete line
    
    for (const line of lines) {
      yield JSON.parse(line);
    }
  }
}
```

## 8. clean up the resources after usage

## 9. use the pooling for database resources

## 10. Monitor the memory usage
```javascript
setInterval(() => {
  const memUsage = process.memoryUsage();
  logger.info('Memory:', {
    rss: Math.round(memUsage.rss / 1024 / 1024) + ' MB',
    heapTotal: Math.round(memUsage.heapTotal / 1024 / 1024) + ' MB',
    heapUsed: Math.round(memUsage.heapUsed / 1024 / 1024) + ' MB',
  });
}, 60000);
```

## 11. Implement proper health checks

```javascript
app.get('/health', (req, res) => {
  const health = {
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    timestamp: new Date(),
  };
  res.json(health);
});

app.get('/ready', async (req, res) => {
  try {
    // Check critical dependencies
    await database.query('SELECT 1');
    await redisClient.ping();
    res.json({ ready: true });
  } catch (error) {
    res.status(503).json({ ready: false, error: error.message });
  }
});
```

## 12. Load and validate environment variables at startup

```javascript
import dotenv from 'dotenv';

dotenv.config();

const config = {
  port: parseInt(process.env.PORT || '3000', 10),
  environment: process.env.NODE_ENV || 'development',
  database: {
    host: process.env.DB_HOST,
    port: parseInt(process.env.DB_PORT || '5432', 10),
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    expiration: process.env.JWT_EXPIRATION || '24h',
  },
};

// Validate required variables
const required = ['DB_HOST', 'DB_USER', 'DB_PASSWORD', 'JWT_SECRET'];
required.forEach(envVar => {
  if (!process.env[envVar]) {
    throw new Error(`Missing required environment variable: ${envVar}`);
  }
});

export default config;
```

## 13. do the logging configuration as per the environment.
```javascript
const getConfig = (env) => {
  const envConfigs = {
    development: { debug: true, logLevel: 'debug' },
    staging: { debug: false, logLevel: 'info' },
    production: { debug: false, logLevel: 'warn' },
  };
  return { ...baseConfig, ...envConfigs[env] };
};
```
## 14. use RequestId in headers and add that in logger for easy tracking of your flow also pass request Id when you call external apis 

```javascript
// ✅ DO: Add request ID middleware
import { v4 as uuidv4 } from 'uuid';
import { AsyncLocalStorage } from 'async_hooks';

export const requestIdStorage = new AsyncLocalStorage();

export function requestIdMiddleware(req, res, next) {
  const requestId = req.headers['x-request-id'] || uuidv4();
  
  requestIdStorage.run(requestId, () => {
    res.setHeader('x-request-id', requestId);
    next();
  });
}

// ✅ DO: Include request ID in all logs
export function getLogger(module) {
  return {
    info: (message, data) => {
      const requestId = requestIdStorage.getStore();
      console.log(JSON.stringify({
        timestamp: new Date(),
        level: 'info',
        requestId,
        module,
        message,
        data,
      }));
    },
    error: (message, error) => {
      const requestId = requestIdStorage.getStore();
      console.error(JSON.stringify({
        timestamp: new Date(),
        level: 'error',
        requestId,
        module,
        message,
        error: {
          message: error.message,
          stack: error.stack,
        },
      }));
    },
  };
}

// ✅ DO: Pass request ID to external API calls
async function callExternalAPI(endpoint) {
  const requestId = requestIdStorage.getStore();
  
  try {
    const response = await axios.post(endpoint, {}, {
      headers: {
        'x-request-id': requestId,
        'x-correlation-id': requestId,
      },
    });
    return response.data;
  } catch (error) {
    logger.error('API call failed', error);
    throw error;
  }
}
```








