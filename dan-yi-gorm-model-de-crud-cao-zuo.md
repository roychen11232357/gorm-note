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

### 指定排除特定欄位(負向表列, 黑名單)

```
user := User{FirstName: "Roy", LastName: "Chen"}
result := db.Omit("FirstName", "LastName", "CreatedAt", "UpdatedAt").Create(&user)
```

產生的sql

{% code overflow="wrap" %}
```
INSERT INTO "gorm_test"."user" ("deleted_at") VALUES (NULL) RETURNING "id"
```
{% endcode %}

所以用omit是可以濾掉gorm.Mode那幾個欄位("CreatedAt", "UpdatedAt")的, 這跟網路上一些舊版資料也有差異

### 批量寫入

```
users := []User{
	{FirstName: "Roy1", LastName: "Chen"},
	{FirstName: "Roy2", LastName: "Chen"},
	{FirstName: "Roy3", LastName: "Chen"},
}

result := db.Create(&users)
```

產生的sql

{% code overflow="wrap" %}
```
INSERT INTO "gorm_test"."user" ("created_at","updated_at","deleted_at","first_name","last_name") VALUES ('2022-09-26 14:19:13.18','2022-09-26 14:19:13.18',NULL,'Roy1','Chen'),('2022-09-26 14:19:13.18','2022-09-26 14:19:13.18',NULL,'Roy2','Chen'),('2022-09-26 14:19:13.18','2022-09-26 14:19:13.18',NULL,'Roy3','Chen') RETURNING "id"
```
{% endcode %}

可以看到, 是有RETURNING "id" 這個東西, 代表就算是batch insert, 還是會有id的喔

### 分批批量寫入

上面的批量寫入實際上是塞在同一個sql裡面, 但如果量大時, 可能會遇到語法上限, 或是效能問題, 這是把同一批insert切成多批是一個比較好的做法

```
result := db.CreateInBatches(&users, 1)
```

## READ

### 查一筆資料

#### First

```
result := db.First(&user)

if result.Error != gorm.ErrRecordNotFound {
	fmt.Println(user)
}
```

產生

{% code overflow="wrap" %}
```
SELECT * FROM "gorm_test"."user" WHERE "user"."deleted_at" IS NULL ORDER BY "user"."id" LIMIT 1
```
{% endcode %}

#### Last

```
result := db.Last(&user)
```

產生

{% code overflow="wrap" %}
```
SELECT * FROM "gorm_test"."user" WHERE "user"."deleted_at" IS NULL ORDER BY "user"."id" DESC LIMIT 1
```
{% endcode %}

#### Take

```
result := db.Take(&user)
```

產生

```
SELECT * FROM "gorm_test"."user" WHERE "user"."deleted_at" IS NULL LIMIT 1
```

可以發現上面3種只取一筆的操作, 會拋出gorm.ErrRecordNotFound, 畢竟是只取1筆, 並不是一個清單, 這樣設計也是合理

### 查多筆資料

```
var users []User

result := db.Find(&users)
```

有一點要注意的, 因為是返回列表, 所以gorm設計上, 即便是空slice, 也不會拋出ErrRecordNotFound
