---
title: MongoDB for Java 
categories: -技术
tags: 
- MongoDb 
- Java
- Spring
date: 2022-01-08 23:42
---

# MongoDB for Java Spring

## 数据类型BSON Types

| Type                       | Number | Alias                 | Notes                      |
| :------------------------- | :----- | :-------------------- | :------------------------- |
| Double                     | 1      | "double"              |                            |
| String                     | 2      | "string"              |                            |
| Object                     | 3      | "object"              |                            |
| Array                      | 4      | "array"               |                            |
| Binary data                | 5      | "binData"             |                            |
| Undefined                  | 6      | "undefined"           | Deprecated.                |
| ObjectId                   | 7      | "objectId"            |                            |
| Boolean                    | 8      | "bool"                |                            |
| Date                       | 9      | "date"                |                            |
| Null                       | 10     | "null"                |                            |
| Regular Expression         | 11     | "regex"               |                            |
| DBPointer                  | 12     | "dbPointer"           | Deprecated.                |
| JavaScript                 | 13     | "javascript"          |                            |
| Symbol                     | 14     | "symbol"              | Deprecated.                |
| JavaScript code with scope | 15     | "javascriptWithScope" | Deprecated in MongoDB 4.4. |
| 32-bit integer             | 16     | "int"                 |                            |
| Timestamp                  | 17     | "timestamp"           |                            |
| 64-bit integer             | 18     | "long"                |                            |
| Decimal128                 | 19     | "decimal"             | New in version 3.4.        |
| Min key                    | -1     | "minKey"              |                            |
| Max key                    | 127    | "maxKey"              |                            |

## 注解概述

MappingMongoConverter 可以使用元数据来驱动对象到文档的映射。下面提供了注释的概述

- `@Id `- 在字段级别应用以标记用于标识目的的字段。
- `@Document`- 应用于类级别以指示该类是映射到数据库的候选对象。您可以指定将存储数据库的集合的名称。
- `@DBRef` - 应用于该字段以指示它将使用 com.mongodb.DBRef 存储。
- `@Indexed` - 应用于字段级别以描述如何索引字段。
- `@CompoundIndex` - 在类型级别应用以声明复合索引
- `@GeoSpatialIndexed` - 在字段级别应用以描述如何对字段进行地理索引。
- `@Transient` - 默认情况下，所有私有字段都映射到文档，此注释不包括应用它的字段存储在数据库中
- `@PersistenceConstructor`- 标记一个给定的构造函数——甚至是一个包保护的构造函数——在从数据库中实例化对象时使用。构造函数参数按名称映射到检索到的 DBObject 中的键值。
- `@Value`- 这个注解是 Spring Framework 的一部分。在映射框架内，它可以应用于构造函数参数。这使您可以使用 Spring 表达式语言语句来转换在数据库中检索到的键的值，然后再将其用于构造域对象。为了引用给定文档的属性，必须使用如下表达式：`@Value("#root.myProperty")`where `root`指的是给定文档的根。
- `@Field` - 在字段级别应用并描述字段的名称，因为它将在 MongoDB BSON 文档中表示，因此允许名称与类的字段名称不同。

一个复杂的映射关系示例

```java
@Document
@CompoundIndexes({
    @CompoundIndex(name = "age_idx", def = "{'lastName': 1, 'age': -1}")
})
public class Person<T extends Address> {

  @Id
  private String id;

  @Indexed(unique = true)
  private Integer ssn;

  @Field("fName")
  private String firstName;

  @Indexed
  private String lastName;

  private Integer age;

  @Transient
  private Integer accountTotal;

  @DBRef
  private List<Account> accounts;

  private T address;

  
  public Person(Integer ssn) {
    this.ssn = ssn;
  }
  
  @PersistenceConstructor
  public Person(Integer ssn, String firstName, String lastName, Integer age, T address) {
    this.ssn = ssn;
    this.firstName = firstName;
    this.lastName = lastName;
    this.age = age;
    this.address = address;
  }

  public String getId() {
    return id;
  }

 // no setter for Id.  (getter is only exposed for some unit testing)

  public Integer getSsn() {
    return ssn;
  }


// other getters/setters ommitted
```

## 复合主键

还支持复合索引。它们是在类级别定义的，而不是在单个属性上定义的。

> 注意：复合索引对于提高涉及多个字段标准的查询的性能非常重要

下面是一个`lastName`以升序和`age`降序创建复合索引的示例 ： 

```java
package com.mycompany.domain;

@Document
@CompoundIndexes({
    @CompoundIndex(name = "age_idx", def = "{'lastName': 1, 'age': -1}")
})
public class Person {

  @Id
  private ObjectId id;
  private Integer age;
  private String firstName;
  private String lastName;

}
```

## 外键引用

Mongodb 存储非关系型文档数据是最佳的，但某些时候关联是有必要的

Mongodb提供了两种关联方式：

1.手动根据id关联，将id存储在文档中，再执行一次查询
2.DBRef，除了_id字段中的值之外，DBRef还包括集合的名称，在某些情况下还包括数据库名称，Java驱动程序提供了很好的支持

以下是 DBRef java示例

```java
@Document
public class Account {

  @Id
  private ObjectId id;
  private Float total;

}

@Document
public class Person {

  @Id
  private ObjectId id;
  @Indexed
  private Integer ssn;
  @DBRef
  private List<Account> accounts;

}
        
```

## Mongodb Spring Converter 

读时从Mongo转换到

```
@ReadingConverter
 public class PersonReadConverter implements Converter<DBObject, Person> {

  public Person convert(DBObject source) {
    Person p = new Person((ObjectId) source.get("_id"), (String) source.get("name"));
    p.setAge((Integer) source.get("age"));
    return p;
  }

}
```

写时转换成MongoDB

```
@WritingConverter
public class PersonWriteConverter implements Converter<Person, DBObject> {

  public DBObject convert(Person source) {
    DBObject dbo = new BasicDBObject();
    dbo.put("_id", source.getId());
    dbo.put("name", source.getFirstName());
    dbo.put("age", source.getAge());
    return dbo;
  }

}
```



[Spring data mongo mapping](https://docs.spring.io/spring-data/data-mongo/docs/1.4.2.RELEASE/reference/html/mapping-chapter.html#mapping-configuration)

[Springboot Data MongoDB指南](https://www.baeldung.com/mongodb-bson)

