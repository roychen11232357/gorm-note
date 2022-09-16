# test

{% code lineNumbers="true" %}
```go
package main

import (
	"fmt"
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
)

func main() {
	host := "localhost"
	user := "postgres"
	password := "admin"
	dbname := "postgres"
	port := "5432"
	applicationName := "gorm-play"

	dsn := fmt.Sprintf("host=%s user=%s password=%s dbname=%s port=%s application_name=%s",
		host,
		user,
		password,
		dbname,
		port,
		applicationName)

	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{
		Logger: logger.Default.LogMode(logger.Info),
	})
	if err != nil {
		fmt.Println(err)
	}

	sqlDB, err := db.DB()
	if err != nil {
		fmt.Println(err)
	}

	sqlDB.SetMaxIdleConns(10)
	sqlDB.SetMaxOpenConns(10)

	sqlDB.Ping()

	select {}
}

```
{% endcode %}

* `sqlDB.Ping() 實際上是執1;`
* application\_name可以對connection做搜尋
* grom的連線池好像只有這兩個func可以使用, 應該沒有一啟動就佔滿連線池的設定, 需要的話要自己實作(跑對應次數的Ping應該可以)
