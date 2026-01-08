## 1. Implement retry with exponential backoff plan
```javascript
async function retryWithBackoff(fn, maxRetries = 3, baseDelay = 1000) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxRetries - 1) throw error;
      
      const delayMs = baseDelay * Math.pow(2, attempt) + Math.random() * 1000;
      await new Promise(resolve => setTimeout(resolve, delayMs));
    }
  }
}

async function fetchUserData(userId) {
  return retryWithBackoff(
    () => axios.get(`https://api.example.com/users/${userId}`),
    3,
    1000
  );
}
```

## 2. use retry for specific error codes especially 408, 429, 500, 502, 503, 504

```javascript
async function callExternalAPI(endpoint) {
  const client = axios.create({
    timeout: 5000,
  });
  
  const retryableErrors = [408, 429, 500, 502, 503, 504];
  
  for (let attempt = 0; attempt < 3; attempt++) {
    try {
      return await client.get(endpoint);
    } catch (error) {
      if (attempt === 2) throw error;
      
      // Don't retry client errors (400, 401, 403, 404)
      if (error.response?.status && !retryableErrors.includes(error.response.status)) {
        throw error;
      }
      
      const delay = Math.pow(2, attempt) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

## 3. make sure always check the rate-limit header for external api calls 

```javascript
async function callRateLimitedAPI(endpoint) {
  try {
    return await axios.get(endpoint);
  } catch (error) {
    if (error.response?.status === 429) {
      const retryAfter = error.response.headers['retry-after'];
      const delayMs = retryAfter ? parseInt(retryAfter) * 1000 : 60000;
      await sleep(delayMs);
      return axios.get(endpoint);
    }
    throw error;
  }
}
```

## 4. cache responses when possible into redis 

## 5. do refresh the jwt token before it expires
```javascript
  async getValidToken() {
    if (this.accessToken && Date.now() < this.expiresAt - 60000) {
      return this.accessToken;
    }
    
    return this.refreshToken();
  }
```
