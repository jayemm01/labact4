# Lab CRUD API

## Setup

Install dependencies:
   ```sh
   npm i
   ```

Open http://localhost:3000/api/health to confirm the server and DB connection.

Create Tables in a Database
   ```
   -- Profiles: optional 1:1 with users (some users may have none)
CREATE TABLE IF NOT EXISTS profiles (
  id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  phone VARCHAR(30),
  city VARCHAR(80),
  country VARCHAR(80),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_profiles_user FOREIGN KEY (user_id) REFERENCES users(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Roles + user_roles: many-to-many between users and roles
CREATE TABLE IF NOT EXISTS roles (
  id INT AUTO_INCREMENT PRIMARY KEY,
  role_name VARCHAR(40) UNIQUE NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS user_roles (
  user_id INT NOT NULL,
  role_id INT NOT NULL,
  assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY(user_id, role_id),
  CONSTRAINT fk_ur_user FOREIGN KEY (user_id) REFERENCES users(id),
  CONSTRAINT fk_ur_role FOREIGN KEY (role_id) REFERENCES roles(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Login audit: many sign-ins per user
CREATE TABLE IF NOT EXISTS login_audit (
  id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  ip_address VARCHAR(45),
  success TINYINT(1) DEFAULT 1,
  occurred_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_la_user FOREIGN KEY (user_id) REFERENCES users(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Self-join example: referrals among users
CREATE TABLE IF NOT EXISTS referrals (
  id INT AUTO_INCREMENT PRIMARY KEY,
  referrer_user_id INT NOT NULL,
  referred_user_id INT NOT NULL,
  referred_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_ref_referrer FOREIGN KEY (referrer_user_id) REFERENCES users(id),
  CONSTRAINT fk_ref_referred FOREIGN KEY (referred_user_id) REFERENCES users(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
   ```

Insert seed data (adjust user IDs to your data):
   ```
   INSERT INTO roles (role_name) VALUES ('student'), ('instructor'), ('admin')
ON DUPLICATE KEY UPDATE role_name = VALUES(role_name);

INSERT IGNORE INTO user_roles (user_id, role_id) VALUES
  (1, 1), (1, 3), (2, 1), (3, 2), (4, 1);

INSERT INTO profiles (user_id, phone, city, country) VALUES
  (1, '09171234567', 'Manila', 'PH'),
  (3, '09981234567', 'Quezon City', 'PH');

INSERT INTO login_audit (user_id, ip_address, success) VALUES
  (1, '192.168.1.10', 1),
  (1, '192.168.1.11', 1),
  (2, '10.0.0.3', 0),
  (3, '172.16.5.5', 1);

INSERT INTO referrals (referrer_user_id, referred_user_id) VALUES
  (1, 2), (1, 3), (3, 4);
   ```
Open phpMyAdmin (SQL tab) or the MySQL CLI. Use the commented templates below and complete the SQL by replacing ... per the guidance. Do not paste literal ellipses—write the proper columns and JOINs.
A) INNER JOIN — Users who have at least one role
   ```
-- EXPECTED COLUMNS: u.id, u.email, r.role_name
SELECT /* u.id, u.email, r.role_name */
FROM /* users u */
INNER JOIN /* user_roles ur ON ur.user_id = u.id */
INNER JOIN /* roles r ON r.id = ur.role_id */
ORDER BY /* u.id, r.role_name */;
   ```
B) LEFT JOIN — All users (with profile info if present)
   ```
-- EXPECTED COLUMNS: u.id, u.email, p.phone, p.city, p.country
SELECT /* u.id, u.email, p.phone, p.city, p.country */
FROM /* users u */
LEFT JOIN /* profiles p ON p.user_id = u.id */
ORDER BY /* u.id */;
   ```
C) RIGHT JOIN — Keep all roles even if unassigned
   ```
-- EXPECTED COLUMNS: r.role_name, user_id, email
SELECT /* r.role_name, u.id AS user_id, u.email */
FROM /* users u */
RIGHT JOIN /* user_roles ur ON ur.user_id = u.id */
RIGHT JOIN /* roles r ON r.id = ur.role_id */
ORDER BY /* r.role_name, user_id */;
   ```
D) FULL OUTER JOIN (emulated) — Profiles vs Users
   ```
-- MySQL emulation: LEFT JOIN UNION RIGHT JOIN, watch duplicates
/* SELECT u.id AS user_id, u.email, p.id AS profile_id
   FROM users u
   LEFT JOIN profiles p ON p.user_id = u.id
UNION
   SELECT u.id AS user_id, u.email, p.id AS profile_id
   FROM users u
   RIGHT JOIN profiles p ON p.user_id = u.id
ORDER BY user_id; */
   ```
E) CROSS JOIN — Every user × every role
   ```
-- EXPECTED COLUMNS: user_id, email, role_name
SELECT /* u.id AS user_id, u.email, r.role_name */
FROM /* users u */
CROSS JOIN /* roles r */
ORDER BY /* u.id, r.role_name */;
   ```
F) SELF JOIN — Who referred whom
   ```
-- EXPECTED COLUMNS: referrer_user_id, referrer_email, referred_user_id, referred_email, referred_at
SELECT /* ref.referrer_user_id, u1.email AS referrer_email,
          ref.referred_user_id, u2.email AS referred_email, ref.referred_at */
FROM /* referrals ref */
INNER JOIN /* users u1 ON u1.id = ref.referrer_user_id */
INNER JOIN /* users u2 ON u2.id = ref.referred_user_id */
ORDER BY /* ref.referred_at DESC */;
   ```
G) Bonus — Latest login per user (LEFT JOIN + subquery)
   ```
-- EXPECTED COLUMNS: u.id, u.email, la.ip_address, la.occurred_at
SELECT /* u.id, u.email, la.ip_address, la.occurred_at */
FROM /* users u */
LEFT JOIN /* (subquery named la that returns the latest login per user) la ON la.user_id = u.id */
ORDER BY /* u.id */;
   ```

Add Protected Report Endpoints (Commented Code Templates)
   ```
// const express = require('express');
// const router = express.Router();
// const auth = require('../middleware/auth');
// const reportCtrl = require('../controllers/reportController');

// // EXAMPLE: enable after implementing in controller
// // router.get('/reports/users-with-roles', auth(true), reportCtrl.usersWithRoles);

// // TODO: Add more once implemented in controller:
// // router.get('/reports/users-with-profiles', auth(true), reportCtrl.usersWithProfiles);
// // router.get('/reports/roles-right-join', auth(true), reportCtrl.rolesRightJoin);
// // router.get('/reports/profiles-full-outer', auth(true), reportCtrl.profilesFullOuter);
// // router.get('/reports/user-role-combos', auth(true), reportCtrl.userRoleCombos);
// // router.get('/reports/referrals', auth(true), reportCtrl.referrals);
// // router.get('/reports/latest-login', auth(true), reportCtrl.latestLogin);

// // module.exports = router;
   ```
   ```
// const db = require('../config/db');

// // === INNER JOIN example (implement this one first) ===
// // exports.usersWithRoles = (req, res) => {
// //   const sql = `/* fill with INNER JOIN from Part 3A */`;
// //   db.query(sql, [], (err, rows) => {
// //     if (err) return res.status(500).json({ error: err.message });
// //     return res.json(rows);
// //   });
// // };

// // === LEFT JOIN ===
// // exports.usersWithProfiles = (req, res) => {
// //   const sql = `/* fill with LEFT JOIN from Part 3B */`;
// //   db.query(sql, [], (err, rows) => {
// //     if (err) return res.status(500).json({ error: err.message });
// //     return res.json(rows);
// //   });
// // };

// // === RIGHT JOIN ===
// // exports.rolesRightJoin = (req, res) => {
// //   const sql = `/* fill with RIGHT JOIN from Part 3C */`;
// //   db.query(sql, [], (err, rows) => {
// //     if (err) return res.status(500).json({ error: err.message });
// //     return res.json(rows);
// //   });
// // };

// // === FULL OUTER (UNION) ===
// // exports.profilesFullOuter = (req, res) => {
// //   const sql = `/* fill with UNION of LEFT + RIGHT from Part 3D */`;
// //   db.query(sql, [], (err, rows) => {
// //     if (err) return res.status(500).json({ error: err.message });
// //     return res.json(rows);
// //   });
// // };

// // === CROSS JOIN ===
/* // exports.userRoleCombos = (req, res) => {
     const sql = `/* fill with CROSS JOIN from Part 3E */`;
     db.query(sql, [], (err, rows) => {
       if (err) return res.status(500).json({ error: err.message });
       return res.json(rows);
     });
   };
*/

// // === SELF JOIN (referrals) ===
// // exports.referrals = (req, res) => {
// //   const sql = `/* fill with SELF JOIN from Part 3F */`;
// //   db.query(sql, [], (err, rows) => {
// //     if (err) return res.status(500).json({ error: err.message });
// //     return res.json(rows);
// //   });
// // };

// // === Latest login per user ===
// // exports.latestLogin = (req, res) => {
// //   const sql = `/* fill with LEFT JOIN + subquery from Part 3G */`;
// //   db.query(sql, [], (err, rows) => {
// //     if (err) return res.status(500).json({ error: err.message });
// //     return res.json(rows);
// //   });
// // };
   ```
   ```
// const reportRoutes = require('./routes/reportRoutes');
// // Mount after auth routes:
// // app.use('/api', reportRoutes);
   ```

Test With Postman
   ```
Login to obtain a JWT (reuse Lab 3 environment).
For each /api/reports/... endpoint, set Authorization → Bearer Token = {{token}}.
   ```
