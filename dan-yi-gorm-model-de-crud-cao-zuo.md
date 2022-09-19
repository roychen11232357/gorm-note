# 單一gorm model的CRUD操作

## CREATE

### 寫入一筆資料

```go
user := User{FirstName: "Roy", LastName: "Chen"}
result := db.Create(&user)

fmt.Println(user.ID)
fmt.Println(result.Error)
fmt.Println(result.RowsAffected)
```

打到pg的sql

{% code overflow="wrap" %}
```sql
INSERT INTO "gorm_test"."user" ("created_at","updated_at","deleted_at","first_name","last_name") VALUES ('2022-09-19 11:40:02.125','2022-09-19 11:40:02.125',NULL,'Roy','Chen') RETURNING "id"
```
{% endcode %}

可以看到created\_at, updated\_at, deleted\_at, id都會自動長出來, 因為都是gorm.model的東西, 但有一個比較酷的地方是, `fmt.Println(user.ID)`這個user.ID竟然有東西, 原因是RETURNING "id", 且傳進db.Create的是user指標, 所以gorm自動幫我們加上去的
