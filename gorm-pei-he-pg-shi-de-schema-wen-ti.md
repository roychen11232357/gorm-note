# gorm配合pg時的schema問題

因為pg的階層是 db -> schema -> table, 但gorm似乎沒對pg的schema做很好的支援, 會有兩個問題

* ORM產生的sql缺少schema prefix
* migrate前一定要先手動建立schema

這裡提供一些做法來處理這兩個問題

加上TablePrefix

```go
NamingStrategy: schema.NamingStrategy{
	TablePrefix:   schemaName + ".",
	SingularTable: true,
}
```

用原生sql執行建立schema

```go
db.Exec("CREATE SCHEMA IF NOT EXISTS " + schemaName)
```

完整代碼

```go
package main

import (
	"fmt"
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
	"gorm.io/gorm/schema"
)

func automigrateTest() {
	host := "localhost"
	user := "postgres"
	password := "admin"
	dbname := "postgres"
	port := "5432"
	applicationName := "gorm-play"
	poolSize := 5
	schemaName := "omggggg"

	dsn := fmt.Sprintf("host=%s user=%s password=%s dbname=%s port=%s application_name=%s",
		host,
		user,
		password,
		dbname,
		port,
		applicationName)

	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{
		Logger: logger.Default.LogMode(logger.Info),
		NamingStrategy: schema.NamingStrategy{
			TablePrefix:   schemaName + ".",
			SingularTable: true,
		},
	})
	if err != nil {
		fmt.Println(err)
	}

	sqlDB, err := db.DB()
	if err != nil {
		fmt.Println(err)
	}

	sqlDB.SetMaxIdleConns(poolSize)
	sqlDB.SetMaxOpenConns(poolSize)

	db.Exec("CREATE SCHEMA IF NOT EXISTS " + schemaName)
}
g
```

