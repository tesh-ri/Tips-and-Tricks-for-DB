# 🚀 Express.js Advanced Project Architecture (Full Implementation)

This document is a comprehensive guide and code reference for setting up a production-grade Express.js backend including:

- Logging
- Validation
- PostgreSQL
- Global Exception Handling
- Global Response Handling
- Keycloak Integration
- Base64 Conversion
- Firebase Integration

Each section includes deep explanations, code samples, architecture patterns, and security/performance notes.

---

## 1. 📚 Structured Logging

### Why Logging is Critical in Production

- Tracks user behavior, API usage, and system health.
- Helps in incident analysis and debugging.
- Enables external monitoring systems (like ELK, Loki).

### Comparison of Libraries

| Library   | Features                   | Performance | Format      | Recommended For           |
| --------- | -------------------------- | ----------- | ----------- | ------------------------- |
| `winston` | Custom levels, transports  | Medium      | JSON & Text | Flexible, enterprise apps |
| `morgan`  | HTTP request logging only  | High        | Text        | Dev, simple apps          |
| `pino`    | Structured JSON, very fast | Very High   | JSON only   | High-performance APIs     |

### Winston Setup Example

```ts
// logger.ts
import { createLogger, format, transports } from 'winston';

export const logger = createLogger({
  level: 'info',
  format: format.combine(
    format.timestamp(),
    format.errors({ stack: true }),
    format.json()
  ),
  transports: [new transports.Console()],
});
```

```ts
// Usage
logger.info('App started');
logger.error('Something went wrong', { error });
```

### Best Practices

- Always use JSON format for structured logs.
- Include `reqId` in logs for traceability.
- Avoid logging secrets or passwords.

---

## 2. ✅ Request Validation

### Libraries Compared

| Library             | Type Safety | Ecosystem     | Syntax      | Recommendation        |
| ------------------- | ----------- | ------------- | ----------- | --------------------- |
| `Joi`               | No          | Mature        | Chained     | Great for schemas     |
| `Zod`               | Yes (TS)    | Modern        | Functional  | Great with TypeScript |
| `express-validator` | No          | Express-first | Declarative | Simple use cases      |

### Example: Custom Joi Middleware

```ts
import Joi from 'joi';

const userSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required(),
});

export const validateUser = (req, res, next) => {
  const { error } = userSchema.validate(req.body);
  if (error) return res.status(400).json({ message: error.message });
  next();
};
```

---

## 3. 🐘 PostgreSQL Connection Pooling (using `pg`)

### Connection Config

```ts
import { Pool } from 'pg';

export const db = new Pool({
  user: 'postgres',
  host: 'localhost',
  database: 'mydb',
  password: 'secret',
  port: 5432,
  max: 10, // connection pool size
  idleTimeoutMillis: 30000,
});
```

### Raw Query Example

```ts
const result = await db.query('SELECT * FROM users');
```

### Transactions

```ts
const client = await db.connect();
try {
  await client.query('BEGIN');
  await client.query('UPDATE users SET coins = coins + 10 WHERE id = $1', [uid]);
  await client.query('COMMIT');
} catch (e) {
  await client.query('ROLLBACK');
  throw e;
} finally {
  client.release();
}
```

---

## 4. ⚠️ Global Exception Handling

### Middleware

```ts
// errorHandler.ts
export const errorHandler = (err, req, res, next) => {
  console.error(err);
  res.status(err.status || 500).json({
    status: 'error',
    message: err.message || 'Internal Server Error',
  });
};
```

### Classification

- **Operational Errors:** Expected, handled (e.g., 404, 400)
- **Programmer Errors:** Bugs (e.g., undefined variables)

---

## 5. 📦 Global Response Handling

### Middleware

```ts
export const responseWrapper = (req, res, next) => {
  const oldJson = res.json;
  res.json = (data) => {
    oldJson.call(res, {
      status: 'success',
      data,
      timestamp: new Date(),
    });
  };
  next();
};
```

### Usage

```ts
res.json({ user: userData });
```

---

## 6. 🔐 Keycloak Integration

### Middleware to Protect Routes

```ts
import Keycloak from 'keycloak-connect';
import session from 'express-session';

const memoryStore = new session.MemoryStore();
export const keycloak = new Keycloak({ store: memoryStore });
```

```ts
// Protect route
app.use('/admin', keycloak.protect(), adminRoutes);
```

### Token Validation & Role Check

```ts
keycloak.protect('realm:admin');
```

---

## 7. 📤 Base64 Encoding/Decoding

```ts
// Encode
const encoded = Buffer.from('Hello World').toString('base64');

// Decode
const decoded = Buffer.from(encoded, 'base64').toString('utf8');
```

### Notes

- Use Base64 for small payloads (e.g., images, tokens)
- Avoid for large files (use streaming instead)

---

## 8. 🔥 Firebase Integration

### Admin SDK Setup

```ts
import admin from 'firebase-admin';
import serviceAccount from './service-account.json';

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: 'https://your-app.firebaseio.com'
});
```

### Auth Strategy

```ts
const verifyToken = async (req, res, next) => {
  const token = req.headers.authorization?.split('Bearer ')[1];
  try {
    const decoded = await admin.auth().verifyIdToken(token);
    req.user = decoded;
    next();
  } catch (err) {
    res.status(401).json({ message: 'Invalid token' });
  }
};
```

---

## 🛡 Security Considerations

- Use `helmet`, `rate-limit`, and `cors` middleware
- Never log sensitive data
- Sanitize DB inputs to avoid SQL injection
- Validate JWTs properly

---

## 🧪 Testing Strategy

- Unit tests: `jest`, `supertest`
- Integration tests: test endpoints with mock DB
- Validate tokens with Firebase emulator or mocks

---


