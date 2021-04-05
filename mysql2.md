# MySQL2
## Connect
### Connection
```
const mysql = require('mysql2/promise');

const conf = {user: 'proj', password: '01234', host: 'mydb-server.example.com', port: 3306, database: 'projectdb'};

const conn = await mysql.createConnection(conf);  // Single
await conn.end();
```

### Pool
```
const pool = mysql.createPool({...conf, connectionLimit:30});  // Connection Pooling
const [list] = await pool.query(`SELECT ...`); // use directly

// or

const conn = await pool.getConnection();  // obtain the connection, then use it. 
const [list] = await conn.query(`SELECT ...`);
conn.release(); // Do not forget to release it.
await pool.end();
```

## Results
### SELECT
```
const [list] = await conn.query(`SELECT id, name FROM users WHERE rank = ?`, [4]);
const [list] = await conn.query(`SELECT id, name FROM users WHERE rank IN (?)`, [3,4,5]);

// list = [
//   {id: 10, name: 'John Doe'}, {id: 13, name: 'Jane Doe'}
//
```
### INSERT
```
// insert one
const [result] = await conn.query(`INSERT INTO users (id, name, rank) VALUES (?,?,?)`, [98, 'John Doe II', 2]);

// insert two or more A
const listArray = [ [98, 'John Doe II', 2], [99, 'Jane Doe II', 4] ];
const [result] = await conn.query(`INSERT INTO users (id, name, rank) VALUES ?`, [listArray]);

// insert two or more B
const listObject = [ {id: 98, name: 'John Doe II', rank: 2}, {id: 99, name: 'Jane Doe II', rank: 4} ];
const [result] = await conn.query(`INSERT INTO users SET ?`, listObject);

// result = ResultSetHeader {
//   fieldCount: 0, affectedRows: 2,
//   insertId: 218, info: '',
//   serverStatus: 2, warningStatus: 0
// }
```

### UPDATE
```
const [result] = await conn.query(`UPDATE users SET name = ?, rank = ? WHERE id = ?`, ['John Doe III', 5, 24]);

// UPSERT
const listArray = [ [98, 'John Doe II', 2], [99, 'Jane Doe II', 4] ];
const [result] = await conn.query(`INSERT INTO users (id, name, rank) AS v ON DUPLICATE KEY UPDATE name = v.name, rank = v.rank`, [listArray]);

// result = ResultSetHeader {
//   fieldCount: 0, affectedRows: 1,
//   insertId: 0, 
//   info: '(Rows matched: 1  Changed: 1  Warnings: 0',
//   serverStatus: 2, warningStatus: 0,
//   changedRows: 1
// }
```

### DELETE
```
const [result] = await conn.query(`DELETE FROM users WHERE id = ?`, [4]);

// result = ResultSetHeader {
//   fieldCount: 0, affectedRows: 1,
//   insertId: 0, info: '',
//   serverStatus: 2, warningStatus: 0
// }
```

## Errors
```
try {
  const [result] = await conn.query(...);
} catch (err) {
// err = Error: This socket has been ended by the other party
//    at PromiseConnection.ping (/.../node_modules/mysql2/promise.js:151:22)
//    at main (/.../process-users.js:37:21) {
//  code: 'EPIPE',
//  errno: undefined,
//  sqlState: undefined,
//  sqlMessage: undefined
//}
}
```
