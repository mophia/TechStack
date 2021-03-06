---
description: Go 原生操作 MySQL
---

# Go 原生操作 MySQL CRUD

先输入以下命令，在 MySQL 中建立一张表。

```sql
CREATE TABLE `user` (
    `id` BIGINT(20) NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(20) DEFAULT '',
    `age` INT(11) DEFAULT '0',
    PRIMARY KEY(`id`)
)ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;
```

然后可以查询到 该表；

```sql
show tables;
```

<img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="original">

插入两条数据：

```sql
insert into user (name, age) values("bingo", 22)
```

```sql
insert into user (name, age) values("violette", 23)
```

查询可得到这些数据：

```sql
select * from user;
```

![](<../.gitbook/assets/image (4) (1) (1).png>)



### 单行查询

```go
func (db *DB) QueryRow(query string, args ...interface{}) *Row go
```

```go
// queryRow 查询单条数据
func queryRow() {
	sqlStr := "select id, name, age from user where id =?"
	var u user
	// 非常重要：确保 QueryRow 之后调用 Scan 方法，否则持有的数据库连接不会被释放
	err := db.QueryRow(sqlStr, 1).Scan(&u.id, &u.name, &u.age) // 1 代表获取第一条
	if err != nil {
		fmt.Printf("scan failed, err:%v\n", err)
		return
	}
	fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age)
}
```

查询结果：![](<../.gitbook/assets/image (1) (1) (1).png>)



### 多行查询

```go
// queryMultiRows 查询多条数据
func queryMultiRows() {
	sqlStr := "select id, name, age from user where id > ?"
	rows, err := db.Query(sqlStr, 0)
	if err != nil {
		fmt.Printf("query failed, err: %v\n", err)
		return
	}
	// 非常重要：关闭rows，释放持有的数据库连接
	defer rows.Close()

	// 循环读取结果集中的数据
	for rows.Next() {
		var u user
		err := rows.Scan(&u.id, &u.name, &u.age)
		if err != nil {
			fmt.Printf("scan failed, err:%v\n", err)
			return
		}
		fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age)
	}
}
```

查询结果：![](<../.gitbook/assets/image (5) (1).png>)



### 插入数据

插入、更新、删除操作都使用 `Exec` 方法。

```go
func (db *DB) Exec(query string, args ...interface{}) (Result, error)
```

```go
// insertRow 插入数据
func insertRow() {
	sqlStr := "insert into user(name, age) values (?,?)"
	re, err := db.Exec(sqlStr, "jun", 22)
	if err != nil {
		fmt.Printf("insert failed, err:%v\n", err)
		return
	}
	var newID int64
	newID, err = re.LastInsertId() // 新插入数据的ID
	if err != nil {
		fmt.Printf("get newID failed, err:%v\n", err)
		return
	} else {
		fmt.Printf("insert success, the id is %d.\n", newID)
	}
}
```

插入结果：![](../.gitbook/assets/image.png)



### 更新数据

```go
// 更新数据
func updateRow() {
	sqlStr := "update user set age=? where id = ?"
	re, err := db.Exec(sqlStr, 39, 3)
	if err != nil {
		fmt.Printf("update failed, err:%v\n", err)
		return
	}
	var n int64
	n, err = re.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("`get RowsAffected failed, err:%v\n", err)
		return
	}
	fmt.Printf("update success, affected rows: %d\n", n)
}

```

更新数据的结果：![](<../.gitbook/assets/image (4) (1).png>)



### 删除数据

```go
// deleteRow 删除数据
func deleteRow() {
	sqlStr := "delete from user where id = ?"
	re, err := db.Exec(sqlStr, 2)
	if err != nil {
		fmt.Printf("delete failed, err:%v\n", err)
		return
	}
	var n int64
	n, err = re.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("`get RowsAffected failed, err:%v\n", err)
		return
	}
	fmt.Printf("delete success, affected rows: %d\n", n)
}

```

删除数据的结果：![](<../.gitbook/assets/image (6).png>)



参考文档：[https://www.liwenzhou.com/posts/Go/go\_mysql/](https://www.liwenzhou.com/posts/Go/go\_mysql/)
