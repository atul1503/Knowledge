# PostgreSQL

## Basic Database Operations

**Database** is where you store all your data in organized tables. **Table** is like a spreadsheet with rows and columns. **Row** is a single record, **Column** is a field that stores one type of data.

### Create a database
```sql
CREATE DATABASE myapp;
```

### Connect to a database
```sql
\c myapp;  -- In psql command line
```

### Create a table
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,          -- SERIAL auto-increments, PRIMARY KEY makes it unique
    name VARCHAR(100) NOT NULL,     -- VARCHAR(100) means text up to 100 characters
    email VARCHAR(255) UNIQUE,      -- UNIQUE means no duplicates allowed
    age INTEGER,                    -- INTEGER stores whole numbers
    created_at TIMESTAMP DEFAULT NOW()  -- TIMESTAMP stores date and time, NOW() sets current time
);
```

### Insert data
```sql
INSERT INTO users (name, email, age) VALUES ('John Doe', 'john@example.com', 25);
INSERT INTO users (name, email, age) VALUES 
    ('Jane Smith', 'jane@example.com', 30),
    ('Bob Johnson', 'bob@example.com', 35);  -- Insert multiple rows at once
```

### Select data
```sql
SELECT * FROM users;                          -- Get all columns from all rows
SELECT name, email FROM users;               -- Get specific columns
SELECT * FROM users WHERE age > 25;          -- Filter with WHERE condition
SELECT * FROM users ORDER BY age DESC;       -- Sort by age, descending
SELECT * FROM users LIMIT 10;                -- Get only first 10 rows
```

### Update data
```sql
UPDATE users SET age = 26 WHERE name = 'John Doe';
UPDATE users SET email = 'new@example.com', age = 27 WHERE id = 1;
```

### Delete data
```sql
DELETE FROM users WHERE id = 1;
DELETE FROM users WHERE age < 18;  -- Delete all users under 18
```

## Data Types You'll Use Most

- **INTEGER** or **INT**: Whole numbers like 1, 100, -50
- **SERIAL**: Auto-incrementing integers, perfect for IDs
- **VARCHAR(n)**: Text with maximum length n characters
- **TEXT**: Text with no length limit
- **BOOLEAN**: True/false values
- **TIMESTAMP**: Date and time together
- **DATE**: Just the date (2024-01-15)
- **TIME**: Just the time (14:30:00)
- **DECIMAL(precision, scale)**: Exact decimal numbers. For example, DECIMAL(10,2) stores numbers like 12345.67
- **JSON** or **JSONB**: Store JSON data. JSONB is faster for queries but slower for inserts

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price DECIMAL(10,2),           -- Up to 8 digits before decimal, 2 after
    in_stock BOOLEAN DEFAULT true,
    metadata JSONB,                -- Store flexible data as JSON
    created_at TIMESTAMP DEFAULT NOW()
);
```

## Working with Relationships

**Foreign Key** connects one table to another. It's like saying "this record belongs to that record in another table."

### One-to-Many relationship
```sql
-- One user can have many orders
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),  -- REFERENCES creates foreign key
    total_amount DECIMAL(10,2),
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Many-to-Many relationship
```sql
-- Many products can belong to many categories
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE product_categories (
    product_id INTEGER REFERENCES products(id),
    category_id INTEGER REFERENCES categories(id),
    PRIMARY KEY (product_id, category_id)  -- Composite primary key prevents duplicates
);
```

### Joining tables
```sql
-- Get user info with their orders
SELECT u.name, u.email, o.total_amount, o.status
FROM users u
JOIN orders o ON u.id = o.user_id;  -- JOIN connects tables using foreign key

-- Get products with their categories
SELECT p.name AS product_name, c.name AS category_name
FROM products p
JOIN product_categories pc ON p.id = pc.product_id
JOIN categories c ON pc.category_id = c.id;
```

**JOIN types:**
- **INNER JOIN** (or just **JOIN**): Only returns rows that match in both tables
- **LEFT JOIN**: Returns all rows from left table, even if no match in right table
- **RIGHT JOIN**: Returns all rows from right table, even if no match in left table
- **FULL OUTER JOIN**: Returns all rows from both tables

## Indexes - Making Your Queries Fast

**Index** is like a book's index - it helps PostgreSQL find data quickly without scanning the entire table. Without indexes, PostgreSQL has to check every row (called a **table scan**), which is slow for large tables.

### When to create indexes
Create indexes on columns you use in:
- `WHERE` clauses
- `ORDER BY` clauses
- `JOIN` conditions
- Columns you search frequently

### B-tree Index (Default)
This is the most common index type. Good for equality and range queries.

```sql
-- Create index on email column (for fast user lookups)
CREATE INDEX idx_users_email ON users(email);

-- Create index on multiple columns (composite index)
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Create unique index (prevents duplicates)
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);
```

### Hash Index
Good for equality comparisons only (WHERE column = value). Faster than B-tree for exact matches but can't handle ranges.

```sql
CREATE INDEX idx_users_id_hash ON users USING HASH (id);
```

### GIN Index (Generalized Inverted Index)
Perfect for JSONB, arrays, and full-text search.

```sql
-- Index JSONB data
CREATE INDEX idx_products_metadata ON products USING GIN (metadata);

-- Now you can search JSON efficiently
SELECT * FROM products WHERE metadata @> '{"color": "red"}';
```

### Partial Index
Index only rows that meet certain conditions. Saves space and makes queries faster.

```sql
-- Index only active users
CREATE INDEX idx_users_active ON users(email) WHERE active = true;

-- Index only recent orders
CREATE INDEX idx_orders_recent ON orders(created_at) WHERE created_at > '2024-01-01';
```

### Expression Index
Index the result of a function or expression.

```sql
-- Index lowercase email for case-insensitive searches
CREATE INDEX idx_users_email_lower ON users(LOWER(email));

-- Now this query will use the index
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';
```

### Managing indexes
```sql
-- View all indexes on a table
\d+ users  -- In psql

-- Check index usage
SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'users';

-- Drop an index
DROP INDEX idx_users_email;

-- Rebuild an index (useful after lots of updates)
REINDEX INDEX idx_users_email;
```

## Improving Query Performance

### 1. Use EXPLAIN to understand your queries
`EXPLAIN` shows you how PostgreSQL will execute your query. `EXPLAIN ANALYZE` actually runs the query and shows real timing.

```sql
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'john@example.com';
```

Look for:
- **Seq Scan** (table scan): Bad for large tables, add an index
- **Index Scan**: Good, using an index
- **Nested Loop**: Can be slow with large datasets
- **Hash Join**: Usually faster than nested loops

### 2. Add indexes on frequently queried columns
```sql
-- Before: slow query without index
SELECT * FROM orders WHERE user_id = 123;  -- Table scan

-- Add index
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- After: fast query with index
SELECT * FROM orders WHERE user_id = 123;  -- Index scan
```

### 3. Use LIMIT to avoid loading unnecessary data
```sql
-- Bad: loads all users
SELECT * FROM users ORDER BY created_at DESC;

-- Good: only loads first 20 users
SELECT * FROM users ORDER BY created_at DESC LIMIT 20;
```

### 4. Use specific columns instead of SELECT *
```sql
-- Bad: loads all columns
SELECT * FROM users WHERE age > 25;

-- Good: only loads needed columns
SELECT name, email FROM users WHERE age > 25;
```

### 5. Use proper WHERE conditions
```sql
-- Bad: function on column prevents index usage
SELECT * FROM users WHERE UPPER(name) = 'JOHN';

-- Good: create expression index or use different approach
CREATE INDEX idx_users_name_upper ON users(UPPER(name));
SELECT * FROM users WHERE UPPER(name) = 'JOHN';  -- Now uses index
```

### 6. Optimize JOINs
```sql
-- Make sure JOIN conditions use indexed columns
SELECT u.name, o.total_amount
FROM users u
JOIN orders o ON u.id = o.user_id  -- Both id and user_id should be indexed
WHERE u.active = true;
```

### 7. Use connection pooling
Don't create a new database connection for every request. Use a connection pool like PgBouncer or your application framework's built-in pooling.

## Transactions - Keeping Data Consistent

**Transaction** is a group of database operations that either all succeed or all fail together. It's like a shopping cart - either you buy everything in it, or you buy nothing.

### Basic transaction
```sql
BEGIN;  -- Start transaction
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- Save all changes
```

If something goes wrong:
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- Oh no, an error happened!
ROLLBACK;  -- Undo all changes since BEGIN
```

### Transaction in application code (Node.js example)
```javascript
const client = await pool.connect();

try {
    await client.query('BEGIN');
    
    // Transfer money between accounts
    await client.query('UPDATE accounts SET balance = balance - $1 WHERE id = $2', [100, 1]);
    await client.query('UPDATE accounts SET balance = balance + $1 WHERE id = $2', [100, 2]);
    
    await client.query('COMMIT');
    console.log('Transfer successful');
} catch (error) {
    await client.query('ROLLBACK');
    console.error('Transfer failed:', error);
} finally {
    client.release();
}
```

### Savepoints - Mini-transactions inside transactions
```sql
BEGIN;
UPDATE users SET name = 'John' WHERE id = 1;

SAVEPOINT my_savepoint;  -- Create a savepoint
UPDATE users SET email = 'john@example.com' WHERE id = 1;
-- Oops, this was wrong
ROLLBACK TO SAVEPOINT my_savepoint;  -- Go back to savepoint

UPDATE users SET email = 'john.doe@example.com' WHERE id = 1;  -- Correct email
COMMIT;  -- Save everything since BEGIN
```

### Transaction isolation levels
Controls how much one transaction can see changes from other transactions.

```sql
-- Default isolation level
BEGIN ISOLATION LEVEL READ COMMITTED;

-- Higher isolation (prevents more anomalies but slower)
BEGIN ISOLATION LEVEL REPEATABLE READ;
BEGIN ISOLATION LEVEL SERIALIZABLE;
```

- **READ UNCOMMITTED**: Can see uncommitted changes from other transactions (rarely used)
- **READ COMMITTED**: Can only see committed changes (default)
- **REPEATABLE READ**: Same data throughout the transaction
- **SERIALIZABLE**: Strongest isolation, transactions appear to run one after another

## Advanced PostgreSQL Features

### Window Functions
Perform calculations across rows related to the current row.

```sql
-- Rank users by age
SELECT name, age, RANK() OVER (ORDER BY age DESC) as age_rank
FROM users;

-- Running total of order amounts
SELECT id, total_amount, 
       SUM(total_amount) OVER (ORDER BY created_at) as running_total
FROM orders;
```

### Common Table Expressions (CTEs)
Create temporary named result sets that exist only for the duration of a query.

```sql
-- Find users and their order count
WITH user_orders AS (
    SELECT user_id, COUNT(*) as order_count
    FROM orders
    GROUP BY user_id
)
SELECT u.name, COALESCE(uo.order_count, 0) as total_orders
FROM users u
LEFT JOIN user_orders uo ON u.id = uo.user_id;
```

### JSON Operations
PostgreSQL has excellent JSON support.

```sql
-- Query JSON data
SELECT * FROM products WHERE metadata->>'color' = 'red';
SELECT * FROM products WHERE metadata @> '{"size": "large"}';

-- Update JSON data
UPDATE products SET metadata = metadata || '{"warranty": "2 years"}' WHERE id = 1;
```

## Database Design Best Practices

### 1. Use appropriate data types
```sql
-- Good: specific types
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255),      -- Emails are rarely longer than 255 chars
    age SMALLINT,            -- Ages fit in SMALLINT (saves space)
    salary DECIMAL(10,2),    -- Exact decimal for money
    is_active BOOLEAN        -- Clear true/false
);

-- Bad: everything is TEXT
CREATE TABLE users (
    id TEXT,
    email TEXT,
    age TEXT,
    salary TEXT,
    is_active TEXT
);
```

### 2. Always use PRIMARY KEY
```sql
-- Good: every table has a primary key
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT
);

-- Bad: no primary key (causes replication issues)
CREATE TABLE posts (
    title VARCHAR(200),
    content TEXT
);
```

### 3. Use constraints to enforce data integrity
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,  -- Must be unique and not null
    age INTEGER CHECK (age >= 0 AND age <= 150),  -- Reasonable age range
    status VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active', 'inactive', 'banned'))
);
```

### 4. Create indexes for performance
```sql
-- Index foreign keys
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Index frequently searched columns
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_published_at ON posts(published_at) WHERE published = true;
```

### 5. Use meaningful names
```sql
-- Good: clear, descriptive names
CREATE TABLE user_profiles (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    date_of_birth DATE
);

-- Bad: unclear abbreviations
CREATE TABLE usr_prof (
    id SERIAL PRIMARY KEY,
    uid INTEGER REFERENCES users(id),
    fn VARCHAR(50),
    ln VARCHAR(50),
    dob DATE
);
```

## Common PostgreSQL Commands

### Database management
```sql
-- List all databases
\l

-- Connect to database
\c database_name

-- Show current database
SELECT current_database();
```

### Table management
```sql
-- List all tables
\dt

-- Describe table structure
\d table_name

-- Show table size
SELECT pg_size_pretty(pg_total_relation_size('table_name'));
```

### User management
```sql
-- Create user
CREATE USER myuser WITH PASSWORD 'mypassword';

-- Grant permissions
GRANT SELECT, INSERT, UPDATE ON users TO myuser;
GRANT USAGE, SELECT ON SEQUENCE users_id_seq TO myuser;

-- Create user with database creation privileges
CREATE USER admin WITH PASSWORD 'adminpass' CREATEDB;
```

### Backup and restore
```bash
# Backup database
pg_dump myapp > myapp_backup.sql

# Restore database
psql myapp < myapp_backup.sql

# Backup with compression
pg_dump -Fc myapp > myapp_backup.dump

# Restore compressed backup
pg_restore -d myapp myapp_backup.dump
```

## Performance Monitoring

### Check slow queries
```sql
-- Enable slow query logging in postgresql.conf
-- log_min_duration_statement = 1000  # Log queries taking more than 1 second

-- Find slow queries
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

### Monitor index usage
```sql
-- Check if indexes are being used
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;
```

### Check table statistics
```sql
-- Update table statistics for better query planning
ANALYZE users;
ANALYZE;  -- Analyze all tables
```

This covers the essential PostgreSQL concepts you'll use in most applications. Start with basic operations and gradually move to more advanced features as your application grows. Remember to always test your queries with `EXPLAIN` and add indexes where needed for good performance. 