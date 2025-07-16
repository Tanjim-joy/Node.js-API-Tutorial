# Node.js API Masterclass with MySQL: Step-by-Step Tutorial

This tutorial will guide you through building a RESTful API using Node.js, Express, and MySQL, from beginner to expert level. We’ll create a simple "Notes" application API with CRUD operations, authentication, and advanced features like connection pooling and environment variables.

## Prerequisites
- **Node.js**: Download and install from [nodejs.org](https://nodejs.org). Verify with `node --version` and `npm --version`.
- **MySQL**: Download and install from [mysql.com](https://www.mysql.com/downloads/). Optionally, install MySQL Workbench for GUI management.
- **Text Editor**: Visual Studio Code (or any editor of your choice).
- **Postman**: For testing API endpoints. Download from [postman.com](https://www.postman.com/downloads/).

## Step 1: Set Up Your Development Environment

1. **Install Node.js**:
   - Download the LTS version from [nodejs.org](https://nodejs.org).
   - Install and verify: `node --version` and `npm --version`.

2. **Install MySQL**:
   - Download MySQL Community Server from [mysql.com](https://www.mysql.com/downloads/).
   - Follow the installation instructions for your OS.
   - Set up a root user and password during installation.
   - Optionally, install MySQL Workbench for easier database management.

3. **Create a Project Directory**:
   ```bash
   mkdir notes-api
   cd notes-api
   npm init -y
   ```
   This creates a `package.json` file with default settings.

4. **Install Dependencies**:
   Install Express, MySQL driver, and other necessary packages:
   ```bash
   npm install express mysql2 dotenv
   ```

## Step 2: Set Up MySQL Database

1. **Start MySQL Server**:
   - On Windows/Mac, start MySQL via the installer or XAMPP.
   - On Linux, use: `sudo systemctl start mysql`.

2. **Create a Database and Table**:
   - Open MySQL Workbench or use the MySQL command line:
     ```bash
     mysql -u root -p
     ```
   - Create a database and table for notes:
     ```sql
     CREATE DATABASE notes_app;
     USE notes_app;
     CREATE TABLE notes (
         id INT AUTO_INCREMENT PRIMARY KEY,
         title VARCHAR(255) NOT NULL,
         contents TEXT NOT NULL,
         created TIMESTAMP DEFAULT CURRENT_TIMESTAMP
     );
     ```

3. **Add Sample Data** (optional):
   ```sql
   INSERT INTO notes (title, contents) VALUES
   ('First Note', 'This is my first note.'),
   ('Second Note', 'This is another note.');
   ```

## Step 3: Configure Environment Variables

1. **Create a `.env` File**:
   In the project root, create a `.env` file to store sensitive data:
   ```env
   MYSQL_HOST=localhost
   MYSQL_USER=root
   MYSQL_PASSWORD=your_password
   MYSQL_DATABASE=notes_app
   PORT=3000
   ```

2. **Install `dotenv`**:
   Already installed in Step 1. This package loads environment variables from the `.env` file.

## Step 4: Connect Node.js to MySQL

1. **Create a Database Connection**:
   Create a `db.js` file in a `config` folder:
   ```bash
   mkdir config
   touch config/db.js
   ```

   Add the following code to `config/db.js`:
   ```javascript
   import mysql from 'mysql2/promise';
   import dotenv from 'dotenv';

   dotenv.config();

   const pool = mysql.createPool({
       host: process.env.MYSQL_HOST,
       user: process.env.MYSQL_USER,
       password: process.env.MYSQL_PASSWORD,
       database: process.env.MYSQL_DATABASE
   });

   export default pool;
   ```

   **Note**: We use `mysql2` (supports promises) and connection pooling for better performance in production.

## Step 5: Set Up Express Server

1. **Create the Main Application File**:
   Create `index.js` in the project root:
   ```javascript
   import express from 'express';
   import dotenv from 'dotenv';
   import noteRoutes from './routes/notes.js';

   dotenv.config();

   const app = express();
   const PORT = process.env.PORT || 3000;

   // Middleware
   app.use(express.json());

   // Routes
   app.use('/api/notes', noteRoutes);

   // Start server
   app.listen(PORT, () => {
       console.log(`Server running on port ${PORT}`);
   });
   ```

2. **Create Routes**:
   Create a `routes` folder and add `notes.js`:
   ```bash
   mkdir routes
   touch routes/notes.js
   ```

   Add the following to `routes/notes.js`:
   ```javascript
   import express from 'express';
   import pool from '../config/db.js';

   const router = express.Router();

   // Get all notes
   router.get('/', async (req, res) => {
       try {
           const [rows] = await pool.query('SELECT * FROM notes');
           res.json(rows);
       } catch (err) {
           res.status(500).json({ error: err.message });
       }
   });

   export default router;
   ```

3. **Run the Server**:
   ```bash
   node index.js
   ```
   Visit `http://localhost:3000/api/notes` in Postman to test the GET endpoint. You should see the sample notes if you added them.

## Step 6: Implement CRUD Operations

Update `routes/notes.js` to include full CRUD operations:

```javascript
import express from 'express';
import pool from '../config/db.js';

const router = express.Router();

// Get all notes
router.get('/', async (req, res) => {
    try {
        const [rows] = await pool.query('SELECT * FROM notes');
        res.json(rows);
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

// Get a single note by ID
router.get('/:id', async (req, res) => {
    try {
        const [rows] = await pool.query('SELECT * FROM notes WHERE id = ?', [req.params.id]);
        if (rows.length === 0) {
            return res.status(404).json({ error: 'Note not found' });
        }
        res.json(rows[0]);
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

// Create a new note
router.post('/', async (req, res) => {
    const { title, contents } = req.body;
    if (!title || !contents) {
        return res.status(400).json({ error: 'Title and contents are required' });
    }
    try {
        const [result] = await pool.query('INSERT INTO notes (title, contents) VALUES (?, ?)', [title, contents]);
        res.status(201).json({ id: result.insertId, title, contents });
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

// Update a note
router.put('/:id', async (req, res) => {
    const { title, contents } = req.body;
    if (!title || !contents) {
        return res.status(400).json({ error: 'Title and contents are required' });
    }
    try {
        const [result] = await pool.query('UPDATE notes SET title = ?, contents = ? WHERE id = ?', [title, contents, req.params.id]);
        if (result.affectedRows === 0) {
            return res.status(404).json({ error: 'Note not found' });
        }
        res.json({ id: req.params.id, title, contents });
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

// Delete a note
router.delete('/:id', async (req, res) => {
    try {
        const [result] = await pool.query('DELETE FROM notes WHERE id = ?', [req.params.id]);
        if (result.affectedRows === 0) {
            return res.status(404).json({ error: 'Note not found' });
        }
        res.json({ message: 'Note deleted' });
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

export default router;
```

**Test CRUD Operations in Postman**:
- **GET** `/api/notes`: Retrieve all notes.
- **GET** `/api/notes/:id`: Retrieve a specific note.
- **POST** `/api/notes`: Send `{ "title": "New Note", "contents": "This is a new note" }` to create a note.
- **PUT** `/api/notes/:id`: Send updated `{ "title": "Updated Note", "contents": "Updated content" }` to update a note.
- **DELETE** `/api/notes/:id`: Delete a note by ID.

## Step 7: Add Authentication (JWT)

1. **Install JWT Dependencies**:
   ```bash
   npm install jsonwebtoken bcryptjs
   ```

2. **Create a Users Table**:
   In MySQL, create a `users` table:
   ```sql
   CREATE TABLE users (
       id INT AUTO_INCREMENT PRIMARY KEY,
       username VARCHAR(255) NOT NULL UNIQUE,
       password VARCHAR(255) NOT NULL
   );
   ```

3. **Create User Routes**:
   Create `routes/users.js`:
   ```javascript
   import express from 'express';
   import bcrypt from 'bcryptjs';
   import jwt from 'jsonwebtoken';
   import pool from '../config/db.js';

   const router = express.Router();

   // Register a new user
   router.post('/register', async (req, res) => {
       const { username, password } = req.body;
       if (!username || !password) {
           return res.status(400).json({ error: 'Username and password are required' });
       }
       try {
           const [existing] = await pool.query('SELECT * FROM users WHERE username = ?', [username]);
           if (existing.length > 0) {
               return res.status(400).json({ error: 'Username already exists' });
           }
           const hashedPassword = await bcrypt.hash(password, 10);
           const [result] = await pool.query('INSERT INTO users (username, password) VALUES (?, ?)', [username, hashedPassword]);
           res.status(201).json({ id: result.insertId, username });
       } catch (err) {
           res.status(500).json({ error: err.message });
       }
   });

   // Login
   router.post('/login', async (req, res) => {
       const { username, password } = req.body;
       try {
           const [users] = await pool.query('SELECT * FROM users WHERE username = ?', [username]);
           if (users.length === 0) {
               return res.status(401).json({ error: 'Invalid credentials' });
           }
           const user = users[0];
           const isMatch = await bcrypt.compare(password, user.password);
           if (!isMatch) {
               return res.status(401).json({ error: 'Invalid credentials' });
           }
           const token = jwt.sign({ id: user.id }, 'your_jwt_secret', { expiresIn: '1h' });
           res.json({ token });
       } catch (err) {
           res.status(500).json({ error: err.message });
       }
   });

   export default router;
   ```

4. **Update `index.js`**:
   Add user routes:
   ```javascript
   import express from 'express';
   import dotenv from 'dotenv';
   import noteRoutes from './routes/notes.js';
   import userRoutes from './routes/users.js';

   dotenv.config();

   const app = express();
   const PORT = process.env.PORT || 3000;

   app.use(express.json());
   app.use('/api/notes', noteRoutes);
   app.use('/api/users', userRoutes);

   app.listen(PORT, () => {
       console.log(`Server running on port ${PORT}`);
   });
   ```

5. **Protect Note Routes**:
   Create a middleware for JWT verification in `middleware/auth.js`:
   ```bash
   mkdir middleware
   touch middleware/auth.js
   ```

   Add to `middleware/auth.js`:
   ```javascript
   import jwt from 'jsonwebtoken';

   const auth = (req, res, next) => {
       const token = req.header('Authorization')?.replace('Bearer ', '');
       if (!token) {
           return res.status(401).json({ error: 'Access denied, no token provided' });
       }
       try {
           const decoded = jwt.verify(token, 'your_jwt_secret');
           req.user = decoded;
           next();
       } catch (err) {
           res.status(401).json({ error: 'Invalid token' });
       }
   };

   export default auth;
   ```

6. **Apply Authentication to Notes Routes**:
   Update `routes/notes.js` to require authentication:
   ```javascript
   import express from 'express';
   import pool from '../config/db.js';
   import auth from '../middleware/auth.js';

   const router = express.Router();

   router.get('/', auth, async (req, res) => {
       try {
           const [rows] = await pool.query('SELECT * FROM notes');
           res.json(rows);
       } catch (err) {
           res.status(500).json({ error: err.message });
       }
   });

   // Add auth middleware to other routes (GET/:id, POST, PUT, DELETE) similarly
   router.get('/:id', auth, async (req, res) => { /* ... */ });
   router.post('/', auth, async (req, res) => { /* ... */ });
   router.put('/:id', auth, async (req, res) => { /* ... */ });
   router.delete('/:id', auth, async (req, res) => { /* ... */ });

   export default router;
   ```

7. **Test Authentication**:
   - Register a user: POST `/api/users/register` with `{ "username": "testuser", "password": "testpass" }`.
   - Login: POST `/api/users/login` to get a JWT token.
   - Use the token in the `Authorization` header (`Bearer <token>`) for note routes.

## Step 8: Advanced Features

1. **Connection Pooling**:
   Already implemented in `db.js` using `mysql2/promise` and `createPool`. This ensures efficient handling of multiple concurrent connections.

2. **Error Handling Middleware**:
   Add to `index.js`:
   ```javascript
   app.use((err, req, res, next) => {
       console.error(err.stack);
       res.status(500).json({ error: 'Something went wrong!' });
   });
   ```

3. **Input Validation**:
   Install `express-validator`:
   ```bash
   npm install express-validator
   ```

   Update `routes/notes.js` for POST:
   ```javascript
   import { body, validationResult } from 'express-validator';

   router.post('/', auth, [
       body('title').notEmpty().withMessage('Title is required'),
       body('contents').notEmpty().withMessage('Contents is required')
   ], async (req, res) => {
       const errors = validationResult(req);
       if (!errors.isEmpty()) {
           return res.status(400).json({ errors: errors.array() });
       }
       // Rest of the POST logic
   });
   ```

4. **Logging**:
   Install `winston`:
   ```bash
   npm install winston
   ```

   Create `utils/logger.js`:
   ```javascript
   import winston from 'winston';

   const logger = winston.createLogger({
       level: 'info',
       format: winston.format.combine(
           winston.format.timestamp(),
           winston.format.json()
       ),
       transports: [
           new winston.transports.File({ filename: 'error.log', level: 'error' }),
           new winston.transports.File({ filename: 'combined.log' })
       ]
   });

   if (process.env.NODE_ENV !== 'production') {
       logger.add(new winston.transports.Console({
           format: winston.format.simple()
       }));
   }

   export default logger;
   ```

   Use in `index.js`:
   ```javascript
   import logger from './utils/logger.js';

   app.use((req, res, next) => {
       logger.info(`${req.method} ${req.url}`);
       next();
   });
   ```

## Step 9: Testing and Deployment

1. **Testing with Postman**:
   - Create a Postman collection for all endpoints.
   - Test edge cases (e.g., invalid IDs, missing fields).

2. **Unit Testing**:
   Install `jest` and `supertest`:
   ```bash
   npm install --save-dev jest supertest
   ```

   Create `tests/notes.test.js`:
   ```javascript
   import supertest from 'supertest';
   import app from '../index.js';

   const request = supertest(app);

   describe('Notes API', () => {
       it('should get all notes', async () => {
           const res = await request.get('/api/notes').set('Authorization', 'Bearer your_token');
           expect(res.status).toBe(200);
           expect(Array.isArray(res.body)).toBe(true);
       });
   });
   ```

   Update `package.json`:
   ```json
   "scripts": {
       "start": "node index.js",
       "test": "jest"
   }
   ```

   Run tests: `npm test`.

3. **Deployment**:
   - Use a platform like Heroku, Render, or AWS.
   - Set up environment variables on the platform.
   - Ensure MySQL is hosted (e.g., PlanetScale, AWS RDS).
   - Use `nodemon` for development: `npm install --save-dev nodemon` and add `"dev": "nodemon index.js"` to `package.json`.

## Step 10: Best Practices and Next Steps

- **Security**:
  - Use HTTPS in production.
  - Sanitize inputs to prevent SQL injection (already mitigated by parameterized queries in `mysql2`).
  - Store JWT secrets in `.env`.

- **Performance**:
  - Use indexing in MySQL for frequently queried columns (e.g., `CREATE INDEX idx_title ON notes(title);`).
  - Implement caching with Redis for frequently accessed data.

- **Next Steps**:
  - Add pagination to GET `/api/notes`.
  - Implement rate limiting with `express-rate-limit`.
  - Explore ORMs like Sequelize for easier database interactions.
  - Add Swagger/OpenAPI documentation.

## Final Project Structure
```
notes-api/
├── config/
│   └── db.js
├── middleware/
│   └── auth.js
├── routes/
│   ├── notes.js
│   └── users.js
├── utils/
│   └── logger.js
├── tests/
│   └── notes.test.js
├── .env
├── index.js
├── package.json
```

## Resources
- [MySQL2 Documentation](https://www.npmjs.com/package/mysql2)
- [Express Documentation](https://expressjs.com/)
- [JWT Documentation](https://www.npmjs.com/package/jsonwebtoken)
- [Winston Logging](https://www.npmjs.com/package/winston)

This tutorial provides a solid foundation for building scalable Node.js APIs with MySQL. From here, you can expand with advanced features like transactions, full-text search, or microservices architecture.
