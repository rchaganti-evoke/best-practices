## 1. use proper data types for columns 

## 2. use foreign key constratints 

## 3. always recommended to have always check constraints 

```javascript
price DECIMAL(10, 2) NOT NULL CHECK (price > 0),
```
## 4. normalize the database structure

## 5. use appropriate column constraints 

```javascript
CONSTRAINT valid_status CHECK (status IN ('pending', 'completed', 'failed', 'refunded'))
```
## 6. index the frequently searcheable columns

## 7. create composite indexes for muti-column searches

## 9. index the columns used in order by clause 

## 10. monitor index usage, remove unused indexes.

## 11. Define connection pooling, timeouts, max connections etc., when you connect to postres from node

```javascript
```javascript
import pg from 'pg';

export const pool = new pg.Pool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 20,  // Maximum connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

setInterval(() => {
  logger.info('Pool metrics', {
    total: pool.totalCount,
    idle: pool.idleCount,
    waiting: pool.waitingCount,
  });
}, 60000);
```

### 12. use parametarized queries

```javascript
export async function searchOrders(userId, startDate, endDate, status) {
  const result = await pool.query(
    `SELECT * FROM orders 
     WHERE user_id = $1 
     AND created_at >= $2 
     AND created_at <= $3 
     AND status = $4`,
    [userId, startDate, endDate, status]
  );
  return result.rows;
}
```

## 13. when it comes to update or delete use them in transaction boundaries

```javascript
export async function updateUserTransaction(userId, email, firstName) {
  return transaction(async (client) => {
    await client.query(
      'UPDATE users SET email = $1, first_name = $2, updated_at = CURRENT_TIMESTAMP WHERE id = $3',
      [email, firstName, userId]
    );
  });
}

export async function transaction(callback) {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    const result = await callback(client);
    await client.query('COMMIT');
    return result;
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```

## 14. use orm when you do selects 

```javascript
import { orm } from './orm-setup.js';

export const User = orm.define('User', {
  email: { type: orm.STRING },
  firstName: { type: orm.STRING },
});

// Queries are parameterized automatically
User.findOne({ where: { email: userInput } });
```
## 15. Implement query logging for monitoring 

```javascript
export async function loggedQuery(sql, params) {
  const startTime = Date.now();
  
  try {
    const result = await pool.query(sql, params);
    const duration = Date.now() - startTime;
    
    if (duration > 1000) {  // Log slow queries
      logger.warn('Slow query', {
        duration,
        sql: sql.substring(0, 200),  // Don't log full potentially large query
        rows: result.rowCount,
      });
    }
    
    return result;
  } catch (error) {
    logger.error('Query error', { sql: sql.substring(0, 200), error });
    throw error;
  }
}
```

## 16. select only needed columns 

## 17. use always limit and offset in select queries 

## 18. use eager loading to avoid N+1 problem 

## 19. minotr slow queries at database level 

## 20. Use migration tool (e.g., knex, flyway, alembic)

```javascript
// migrations/001_initial_schema.js
export async function up(knex) {
  await knex.schema.createTable('users', (table) => {
    table.uuid('id').primary().defaultTo(knex.raw('gen_random_uuid()'));
    table.string('email').notNullable().unique();
    table.string('password_hash').notNullable();
    table.string('first_name').notNullable();
    table.string('last_name').notNullable();
    table.boolean('is_active').defaultTo(true);
    table.timestamps(true, true);  // created_at, updated_at
    table.index(['email', 'is_active']);
  });
```










