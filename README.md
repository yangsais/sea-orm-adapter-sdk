# sea orm casbin adapter

## update
    1. the version 3.0 is enabled only `logging` freatrue .
    2. you can use any run_time to run the adapter. but same with the sea-orm's run_time.
```toml
[dependencies]
sea_orm_casbin_adapter = {version = "0.3", features = ["mysql", "runtime-tokio-native-tls"]}
sea-orm = {version = "0.6.0", default-features = false, features = ["sqlx-mysql", "macros", "runtime-tokio-native-tls"]}
```

or

```toml
[dependencies]
sea_orm_casbin_adapter = {version = "0.3", features = ["mysql", "runtime-tokio-native-tls"]}

[dependencies.sea-orm]
default-features = false
features = ["macros", "runtime-tokio-native-tls"]
version = "0.6.0"

[features]
default = ["postgres", "mysql"]
mysql = ["sea-orm/sqlx-mysql"]
postgres = ["sea-orm/sqlx-postgres"]
sqlite = ["sea-orm/sqlx-sqlite"]
```


## Introduction
> sea-orm-casbin-adapter is a casbin adapter for sea-orm.  
> just like sqx-adapter, it can be used in sea-orm.

## test

[testcase/测试例子](https://github.com/lingdu1234/sea_orm_casbin_adapter/blob/main/tests/test.md)

# ...
a adapter with sea-orm. it is only tested on Mysql,Postgresql. not Sqlite3,may be work.

## usage<使用方法>
1.  add dependencies
```toml
[dependencies]
sea_orm_casbin_adapter = "*"
tokio = "*"
```
2.  add casbin model policy file
3.  add code to your code
   ```rust
   use sea_orm_casbin_adapter::{casbin::prelude::*, SeaOrmAdapter};
   
#[tokio::main]
async fn main() -> Result<()> {
    let m = DefaultModel::from_file("config/casbin_conf/rbac_model.conf")
        .await
        .unwrap();

    // mysql://root:youpassword@127.0.0.1:13306/wk
    // postgres://postgres:youpassword@127.0.0.1:25432/wk
    let db = SeaOrmAdapter::new("postgres://postgres:youpassword@127.0.0.1:25432/wk")
        .await
        .expect("open db error");

    let mut e = Enforcer::new(m, db).await?;
    e.enable_log(true);
    e.add_named_policy(
        "g",
        vec![
            "data9_admin".to_string(),
            "data9".to_string(),
            "write".to_string(),
        ],
    )
    .await
    .unwrap();

    e.add_policy(
        vec![
            "dataxx_admin".to_string(),
            "dataxx".to_string(),
            "write".to_string(),
        ],
    )
    .await
    .unwrap();

    e.add_policies(vec![
        vec![
            "data1_admin".to_string(),
            "data4".to_string(),
            "write".to_string(),
        ],
        vec![
            "data3_admin".to_string(),
            "data5".to_string(),
            "write".to_string(),
        ],
    ])
    .await
    .unwrap();

    // e.remove_policies(vec![
    //     vec![
    //         "data6_admin".to_string(),
    //         "data4".to_string(),
    //         "write".to_string(),
    //     ],
    //     vec![
    //         "data6_admin".to_string(),
    //         "data5".to_string(),
    //         "write".to_string(),
    //     ],
    // ])
    // .await
    // .unwrap();
    println!("C===============");
    e.enforce(("data2_admin", "data2", "write"))?;
    println!("D===============");

    println!("e===============");

    Ok(())
}

   ```

   支持直接传入数据库连接
   ```rust
   let m = DefaultModel::from_file("config/casbin_conf/rbac_model.conf")
        .await
        .unwrap();
    let pool: DatabaseConnection = Database::connect("protocol://username:password@host/database").await?;
    let adpt = SeaOrmAdapter::new_with_pool(pool).await.unwrap();
    let mut e = Enforcer::new(m, adpt).await?;
    e.enforce(("data2_admin", "data2", "write"))?;

   ```

