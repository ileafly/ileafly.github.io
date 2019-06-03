
title: 深入浅出CoreStore
date: 2019-01-07 09:59:44 +0800
categories: iOS
---


[CoreStore](https://github.com/JohnEstropia/CoreStore)是一个使用Swift包装CoreData的框架，号称利用Swift语言的特性，优雅而又安全的释放CoreData的真正潜力。

<a name="c666ad11"></a>
### 特性

- 充分利用了Swift语言的优雅和安全的特点
- 安全的并发架构 所有的更新都是通过串行事务完成的
- 清晰、便捷的查询API
- 类型安全，可以方便的配置监听
- 支持高效率的批量导入
- 支持纯Swift代码声明安全的成员变量，无需维护**.xcdatamodeId**文件
- 支持进阶式升级
- 支持自定义升级
- 支持一个**DataStack**管理多个存储对象

<a name="704f29e0"></a>
### 基本用法
<a name="ef56a618"></a>
#### 数据库插入
```swift
CoreStore.perform(
  asynchronous: { (transaction) -> Void in
        let person = transaction.create(Into<MyPersonEntity>())
        person.name = "John Smith"
        person.age = 42
  },
  completion: { (result) -> Void in 
        switch result {
        case .success: print("success!")
        case .failure(let error): print(error)
        }
  }
)
```

<a name="9d876ea2"></a>
#### 数据库查询
```swift
let people = CoreStore.fetchAll(
    From<MyPersonEntity>()
        .where(\.age > 30),
        .orderBy(.ascending(\.name), .descending(.\age)),
        .tweak({ $0.includesPendingChanges = false })
)
```

```swift
let maxAge = CoreStore.queryValue(
    From<MyPersonEntity>()
        .select(Int.self, .maximum(\.age))
)
```

<a name="e1255531"></a>
#### 数据库更新
```swift
CoreStore.perform(asynchronous: { (transaction) -> Void in 
        let predicate = NSPredicate(format: "name = leafly")
        let person = transaction.fetchOne(From<MyPersonEntity>(),
                                          Where<MyPersonEntity>(predicate))
        preson.age = 29
  },
  completion: { (result) -> Void in 
        switch result {
        case .success: print("success!")
        case .failure(let error): print(error)
        }
  }
)
```

<a name="7118290c"></a>
#### 数据库删除
```swift
let predicate = NSPredicate(formate: "age = 30")
CoreStore.perform(asynchronous: { (transaction) -> Void in
     transaction.deleteAll(From<MyPersonEntity>(),
                           Where<MyPersonEntity>(predicate)
 },
 completion: { (result) -> Void in 
        switch result {
        case .success: print("success!")
        case .failure(let error): print(error)
        }
  }
)
```

<a name="f895fc53"></a>
### 数据库升级
```swift
// 创建dataStack 需要指明迁移路径
let dataStack = DataStack(xcodeModelName: "Model", 
                          bundle: Bundle.main, 
                          migrationChain: ["Model", "ModelV2"])
// sqlite的路径
let sqliteFileURL = FileManager.default.urls(for: systemDirectorySearchPath, in: .userDomainMask).first!.appendingPathComponent(Bundle.main.bundleIdentifier, isDirectory: true).appendingPathComponent("Model.sqlite")
// addStorate
_ = dataStack.addStorage(
  SQLiteStore(
    fileURL: sqliteFileURL,
    localStoreageOptions: .recreateStoreOnModelMismatch
  ),
  completion: { (result) -> Void in
               switch result {
                 case .success(let storage):
                    // 做一些数据库创建后的操作
                 case .failure(let error):
                    // 数据库创建失败
               }
              }
)

CoreStore.defaultStack = dataStack
```


