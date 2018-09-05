title: Swift开源项目ObjectMapper实践
tags:
  - Swift
  - iOS进阶
categories:
  - iOS
date: 2017-07-20 17:42:00
---
近期项目打算全面向swift迁移，虽然两三年前有写过swift项目但是很长时间没有开发很多知识点已经模糊，最近打算就热门的几个第三方库的使用方法进行一个调研

今天就先从[ObjectMapper](https://github.com/Hearst-DD/ObjectMapper)入手，ObjectMapper是一个由swift写的json和模型转换的开源库，目前已经有5950个star

### 先从官方文档入手，进行一个简单的介绍

##### 支持的功能
- JSON向模型的转换
- 模型向JSON的转换
- 嵌套结构的解析
- mapping时的自定义转换
- 结构体的支持

##### 基础用法
ObjectMapper中定义了一个协议[Mappable](https://github.com/Hearst-DD/ObjectMapper/blob/master/Sources/Mappable.swift)

Mappable协议中声明了两个方法

```
mutation func mapping(map: Map)

init?(map: Map)
```

我们需要在模型中遵循这个协议，官方给出了一个参考：
```
class User: Mappable {
    var username: String?
    var age: Int?
    var weight: Double!
    var array: [Any]?
    var dictionary: [String : Any] = [:]
    var bestFriend: User?                       // Nested User object
    var friends: [User]?                        // Array of Users
    var birthday: Date?

    required init?(map: Map) {

    }

    // Mappable
    func mapping(map: Map) {
        username    <- map["username"]
        age         <- map["age"]
        weight      <- map["weight"]
        array       <- map["arr"]
        dictionary  <- map["dict"]
        bestFriend  <- map["best_friend"]
        friends     <- map["friends"]
        birthday    <- (map["birthday"], DateTransform())
    }
}

struct Temperature: Mappable {
    var celsius: Double?
    var fahrenheit: Double?

    init?(map: Map) {

    }

    mutating func mapping(map: Map) {
        celsius     <- map["celsius"]
        fahrenheit  <- map["fahrenheit"]
    }
}
```

一旦我们的类或结构体如上面的示例一样实现了协议，我们就可以方便的进行JSON和模型之间的转换

```
let user = User(JSONString: JSONString)

let JSONString = user.toJSONString(prettyPrint: true)
```
当然也可以通过[Mapper](https://github.com/Hearst-DD/ObjectMapper/blob/master/Sources/Mapper.swift)类来进行转换

```
let user = Mapper<User>().map(JSONString: JSONString)

let JSONString = Mapper().toJSONString(user, prettyPrint: true)
```

##### 嵌套对象的映射

正如前面所列，ObjectMapper支持嵌套对象的映射

列如：
```
{
    "distance" : {
        "text" : "102",
        "value" : 31
    }
}
```
我们想要直接取出distance对象中的value值，可以设置如下mapping
```
func mapping(map: Map) {
    distance <- map["distance.value"]
}
```

##### 自定义转换规则

ObjectMapper允许开发者在数据映射过程中指定转换规则

```
class People: Mappable {
   var birthday: NSDate?
   
   required init?(_ map: Map) {
       
   }
   
   func mapping(map: Map) {
       birthday <- (map["birthday"], DateTransform())
   }
   
   let JSON = "\"birthday\":1458117795332"
   let result = Mapper<People>().map(JSON)
}

```
由于我们指定了`birthday`的转换规则，所以上述代码在解析JSON数据的时候会将long类型转换成Date类型

除了使用ObjectMapper给我们提供的转换规则外，我们还可以通过实现[TransformType](https://github.com/Hearst-DD/ObjectMapper/blob/master/Sources/TransformType.swift)协议来自定义我们的转换规则

```
public protocol TransformType {
    typealias Object
    typealias JSON
    
    func transformFromJSON(value: AnyObject?) -> Object?
    func transformToJSON(value: Object?) -> JSON?
}
```
ObjectMapper为我们提供了一个[TransformOf](https://github.com/Hearst-DD/ObjectMapper/blob/master/Sources/TransformOf.swift)类来实现转换结果，TransformOf实际就是实现了TransformType协议的，TransformOf有两个类型的参数和两个闭包参数，类型表示参与转换的数据的类型，闭包表示转换的规则

```
let transform = TransformOf<Int, String>(fromJSON: { (value: String?) -> Int? in 
}, toJSON: { (value: Int?) -> String? in 
  // transform value from Int? to String?
  if let value = value {
      return String(value)
  }
  return nil
})

id <- (map["id"], transform)
```

##### 泛型对象

ObjectMapper同样可以处理泛型类型的参数，不过这个泛型类型需要在实现了Mappable协议的基础上才可以正常使用

```
class User: Mappable {
    var name: String?
    
    required init?(_ map: Map) {
        
    }
    
    func mapping(_ map: Map) {
        name <- map["name"]
    }
}

class Result<T: Mappable>: Mappable {
    var result: T?
    
    required init?(_ map: Map) {
        
    }
    
    func mapping(map: Map) {
        result <- map["result"]
    }
}

let JSON = "{\"result\": {\"name\": \"anenn\"}}"
let result = Mapper<Result<User>>().map(JSON)
```

### 原理解析

ObjectMapper声明了一个Mappable协议，这个协议里声明了两个方法
```
init?(map: Map)

mutating func mapping(map: Map)
```
并且通过对Mappable协议的扩展，增加了JSON和模型之间的四个转换方法
```
public extension BaseMappable {
    
    /// Initializes object from a JSON String
    public init?(JSONString: String, context: MapContext? = nil) {
        if let obj: Self = Mapper(context: context).map(JSONString: JSONString) {
            self = obj
        } else {
            return nil
        }
    }
    
    /// Initializes object from a JSON Dictionary
    public init?(JSON: [String: Any], context: MapContext? = nil) {
        if let obj: Self = Mapper(context: context).map(JSON: JSON) {
            self = obj
        } else {
            return nil
        }
    }
    
    /// Returns the JSON Dictionary for the object
    public func toJSON() -> [String: Any] {
        return Mapper().toJSON(self)
    }
    
    /// Returns the JSON String for the object
    public func toJSONString(prettyPrint: Bool = false) -> String? {
        return Mapper().toJSONString(self, prettyPrint: prettyPrint)
    }
}

```
我们在使用ObjectMapper进行转换时，首先需要让模型遵循Mappable协议，并且实现Mappable声明的两个方法

```
class User: Mappable {
    var username: String?
    var age: Int?

    required init?(map: Map) {

    }

    // Mappable
    func mapping(map: Map) {
        username    <- map["username"]
        age         <- map["age"]
    }
}
```
因为ObjectMapper为Mappable增加了四个转换方法，所以User也继承了这四个转换方法

```
let user = User(JSONString: JSONString)
```
这个方法的调用其实就是帮助我们执行了
```
Mapper().map(JSONString: JSONString)
```
因此，我们也可以直接通过如下方法进行转换
```
let user = Mapper<User>().map(JSONString: JSONString)
```
接下来就是ObjectMapper的核心啦~
为什么通过Mapper类能完成转换呢？

我们来逐步了解[Mapper](https://github.com/Hearst-DD/ObjectMapper/blob/master/Sources/Mapper.swift)的实现原理

```
public final class Mapper<N: BaseMappable> {
    
}
```
正如上面的代码块所示，首先Mapper类定义了一个泛型N，N必须要遵循Mappable协议，也就是我们的模型User
继续往下看，我们选择一个常用的方法
```
public func map(JSONString: String) -> N? {
        if let JSON = Mapper.parseJSONStringIntoDictionary(JSONString: JSONString) {
            return map(JSON: JSON)
        }
        
        return nil
    }
```
没错，我们前面写的Mapper的转换方法正是使用的这个方法
```
Mapper().map(JSONString: JSONString)
```
那我们就来一层一层详细分析一下这个方法的内部实现吧

```
// 1. 调用Mapper的静态方法parseJSONStringIntoDictionary来将JSON转换成字典
// 2. 调用map方法将转换后的字典转换成模型并返回
if let JSON = Mapper.parseJSONStringIntoDictionary(JSONString: JSONString) {
            return map(JSON: JSON)
        }
        
        return nil
```
先分解`parseJSONStringIntoDictionary`的实现

```
/// Convert a JSON String into a Dictionary<String, Any> using NSJSONSerialization
    public static func parseJSONStringIntoDictionary(JSONString: String) -> [String: Any]? {
        let parsedJSON: Any? = Mapper.parseJSONString(JSONString: JSONString)
        return parsedJSON as? [String: Any]
    }

    /// Convert a JSON String into an Object using NSJSONSerialization
    public static func parseJSONString(JSONString: String) -> Any? {
        let data = JSONString.data(using: String.Encoding.utf8, allowLossyConversion: true)
        if let data = data {
            let parsedJSON: Any?
            do {
                parsedJSON = try JSONSerialization.jsonObject(with: data, options: JSONSerialization.ReadingOptions.allowFragments)
            } catch let error {
                print(error)
                parsedJSON = nil
            }
            return parsedJSON
        }

        return nil
    }

```
通过上述代码的逻辑不难发现，`parseJSONStringIntoDictionary`就是利用了系统的`JSONSerialization`将JSON字符串转换成字典

理解了第一步，接下来我们就来看看第二步的实现原理

我们先整体看一下map方法的实现代码

```
/// Maps a JSON dictionary to an object that conforms to Mappable
    public func map(JSON: [String: Any]) -> N? {
        let map = Map(mappingType: .fromJSON, JSON: JSON, context: context, shouldIncludeNilValues: shouldIncludeNilValues)
        
        if let klass = N.self as? StaticMappable.Type { // Check if object is StaticMappable
            if var object = klass.objectForMapping(map: map) as? N {
                object.mapping(map: map)
                return object
            }
        } else if let klass = N.self as? Mappable.Type { // Check if object is Mappable
            if var object = klass.init(map: map) as? N {
                object.mapping(map: map)
                return object
            }
        } else if let klass = N.self as? ImmutableMappable.Type { // Check if object is ImmutableMappable
            do {
                return try klass.init(map: map) as? N
            } catch let error {
                #if DEBUG
                let exception: NSException
                if let mapError = error as? MapError {
                    exception = NSException(name: .init(rawValue: "MapError"), reason: mapError.description, userInfo: nil)
                } else {
                    exception = NSException(name: .init(rawValue: "ImmutableMappableError"), reason: error.localizedDescription, userInfo: nil)
                }
                exception.raise()
                #else
                NSLog("\(error)")
                #endif
            }
        } else {
            // Ensure BaseMappable is not implemented directly
            assert(false, "BaseMappable should not be implemented directly. Please implement Mappable, StaticMappable or ImmutableMappable")
        }
        
        return nil
    }

```
我们先忽略掉`StaticMappable`和`ImmutableMappable`这两种协议的处理逻辑，直接关注最重要的`Mappable`协议的实现

```
// 根据传入的JSON字典等数据创建一个map对象

let map = Map(mappingType: .fromJSON, JSON: JSON, context: context, shouldIncludeNilValues: shouldIncludeNilValues)

// 判断要转换成的模型是不是 遵循的Mappable协议
if let klass = N.self as? Mappable.Type 

// 创建一个N类型的对象
var object = klass.init(map: map) as? N

// 获取模型中定义的解析规则，完成解析
object.mapping(map: map)

// 返回生成好的模型
return object
```
这里还留了一个疑问，为何执行完`object.mapping(map: map)`后，模型就能完成解析呢

继续分析[Map]((https://github.com/Hearst-DD/ObjectMapper/blob/master/Sources/Mapper.swift)类

原来在Map类中使用了`subscript`来自定义下标
同样我们直接来分析最重要的那个自定义下标的方法
```
public subscript(key: String, nested nested: Bool, delimiter delimiter: String, ignoreNil ignoreNil: Bool) -> Map {
        // save key and value associated to it
        currentKey = key
        keyIsNested = nested
        nestedKeyDelimiter = delimiter
        
        if mappingType == .fromJSON {
            // check if a value exists for the current key
            // do this pre-check for performance reasons
            if nested == false {
                let object = JSON[key]
                let isNSNull = object is NSNull
                isKeyPresent = isNSNull ? true : object != nil
                currentValue = isNSNull ? nil : object
            } else {
                // break down the components of the key that are separated by .
                (isKeyPresent, currentValue) = valueFor(ArraySlice(key.components(separatedBy: delimiter)), dictionary: JSON)
            }
            
            // update isKeyPresent if ignoreNil is true
            if ignoreNil && currentValue == nil {
                isKeyPresent = false
            }
        }
        
        return self
    }

```

不难发现，在这方法中，我们从JSON字典中根据key获取了value，原来当我们调用`object.mapping(map: map)`时，就会依次根据我们配置的mapping值获取对应的value

#### ObjectMapper实践

分析完原理，我们还需要能够熟练的运用ObjectMapper来帮助我们完成解析功能，ObjectMapper能够帮助我们处理数据的方法有很多，这里我就先简单跟大家分享几种在项目中常用的方法，我已经将相关的[Demo](https://github.com/luzhiyongGit/ObjectMapperDemo/tree/master)上传到github上，欢迎大家star

###### 解析单一结构的模型
单一结构的模型解析比较简单，大家了解一下即可
```
{
  "name": "objectmapper",
  "age": 18,
  "nickname": "mapper",
  "job": "swifter"
}
```
当我们拿到如上所示的json数据时，只需要创建一个遵循`Mappable`的模型，并配置好解析路径即可
```
class User: Mappable {

    var name: String?
    var age: Int?
    var nickname: String?
    var job: String?
    
    required init?(map: Map) {
        
    }
    
    func mapping(map: Map) {
        name <- map["name"]
        age <- map["age"]
        nickname <- map["nickname"]
        job <- map["job"]
    }
    
}

// 使用方法
let user = Mapper<User>().map(JSONString: json.rawString()!)

```
###### 解析模型中嵌套模型的情况
有时候我们拿到的json数据嵌套了好几层的结构，而我们刚好也需要逐层解析拿到每个模型的数据，下面我就通过一个两层结构给大家演示一下处理的方法

```
{
  "weather": "sun",
    "temperature": {
        "celsius": 70,
        "fahrenheit": 34
    },
}
```
正如上面的json数据所示，我们需要创建一个weather模型，同时包含一个temperature模型来解析json数据，这时我们就需要运用泛型来帮助我们达到嵌套的目的
```
class Weather<T: Mappable>: Mappable {

    var weather: String?
    var temperature: T?
    
    required init?(map: Map) {
        
    }
    
    func mapping(map: Map) {
        weather <- map["weather"]
        temperature <- map["temperature"]
    }
}

class Temperature: Mappable {
    
    var celsius: Double?
    var fahrenheit: Double?
    
    required init?(map: Map) {
        
    }
    
    func mapping(map: Map) {
        celsius <- map["celsius"]
        fahrenheit <- map["fahrenheit"]
    }
}

// 使用方法如下
let weather = Mapper<Weather<Temperature>>().map(JSONString: json.rawString()!)

```
###### 解析模型中嵌套模型但是我们只需要拿到子模型的属性值

有些json数据嵌套的内容我们不需要分模型来获取，只需要将属性统一到一个模型中使用
```
{
    "distance": {
        "text": "102 ft",
        "value": 31
    }
}
```
```
class Distance: Mappable {
    var text: String?
    var value: Int?
    
    required init?(map: Map) {
        
    }
    
    func mapping(map: Map) {
        text <- map["distance.text"]
        value <- map["distance.value"]
    }

}
```
####### 解析指定的一个数组
```
{
    "status": "200",
    "msg": "success",
    "features": [
       {
         "name": "json解析"
       },
       {
         "name": "模型转换"
       },
       {
         "name": "数据处理"
       },
       {
         "name": "mapper进阶"
       }
    ]
}
```
通过我们拿到的返回结果如上面的代码段所示，features是我们真正关心的数据，它是一个数组结构，我们期望能够得到的是一个数组，里面包含若干个feature模型

首先我们必不可少的就是要创建一个Feature模型
```
class Feature: Mappable {

    var name: String?
    
    required init?(map: Map) {
        
    }
    
    func mapping(map: Map) {
        name <- map["name"]
    }
    
}
```
在解析这个JSON数据时，我们可以忽略掉其他信息，先获取features对应的json数据
```
let featureJson = json["features"];
```
然后只需要使用Mapper的高级用法`mapArray`方法即可直接得到数组对象
```
let features = Mapper<Feature>().mapArray(JSONString: featureJson.rawString()!)
```

ObjectMapper的高级用法还有很多，掌握上面的这些用法基本已经可以在项目中使用ObjectMapper达到我们的需求了，后面有时间我会在此博客和[Demo](https://github.com/luzhiyongGit/ObjectMapperDemo/tree/master)基础上更新更多的用法供大家参考



