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

可以看到created\_at, updated\_at, deleted\_at, id都會自動長出來, 因為都是gorm.model的東西, 但有一個比較酷的地方是, `fmt.Println(user.ID)`這個user.ID竟然有東西, 原因是RETURNING "id", 且傳進db.Create的是user指標, 所以gorm自動幫我們加上去的\


### 指定寫入特定欄位(正向表列, 白名單)

```
user := User{FirstName: "Roy", LastName: "Chen"}
result := db.Select("FirstName").Create(&user)
```

產生的sql

{% code overflow="wrap" %}
```
INSERT INTO "gorm_test"."user" ("created_at","updated_at","first_name") VALUES ('2022-09-26 09:49:15.997','2022-09-26 09:49:15.997','Roy') RETURNING "id"
```
{% endcode %}

可以發現LastName就沒寫進去了, 但這裡有一點跟網路上其他文章不太一樣, 有些文章提到, 如果是用這種部分欄位寫入的方式, 會繞過gorm預設的create\_at, update\_at, 但我實測後還是會有這些值喔, 可能是版本的差異



