---
layout: default
title: 3306 | MYSQL
nav_order: 5
---

# 3306 | MYSQL

---

## Connect to Remote MySQL Server

```bash
mysql -h 192.71.145.3 -u root
```

### Show Databases

```sql
show databases;
```

### Use Table and Select Records

```sql
use books;
select count(*) from authors;
```
