---
title: WebFlux中mongo操作-Transaction
comments: true
categories: [程序猿]
tags: [Java, Reactive, 事务, mongo, 数据库, Flux, mongo]
toc: true
visible: show
indexing: true
date: 2020-01-17 15:16:46
---
<br>
<!--more-->
```java
@PostMapping("/test")
public Mono testA(@RequestParam boolean exception) {
  return embedService.saveAC(new ADocument("张三"), new CDocument("李四"), exception);
}

@Override
public Mono<Boolean> saveAC(ADocument aDocument, CDocument cDocument, boolean exception) {
  return reactiveMongoTemplate.inTransaction()
    //所有文档的持久化操作都只能在单独一个execute函数中汇总实现
    .execute(action -> action.insert(aDocument)
             .flatMap(a -> {
               cDocument.setName(a.getName() + "copy");
               return action.insert(cDocument)
                 .map(d -> {
                   if (exception) {
                     //测试跨文档的异常回滚
                     throw Exceptions.propagate(new RuntimeException("模拟异常的出现"));
                   }
                   return d;
                 });
             })
            )
    //如果里面是个mono，则用next取出第一个元素就是里面的mono
    .next()
    .map(list -> {
      //需要注意，在execute之外的函数中产生的异常，不会触发事务的回滚。
      //                    if (exception) {
      //                        throw Exceptions.propagate(new RuntimeException("模拟异常的出现"));
      //                    }
      return Boolean.TRUE;
    });
}
```

flux的数据库操作，在有事务的前提下不能用flatMap，要用事务不能用flatMap要用concatMap保持有序

```java
@PostMapping("/test")
public Mono testA(@RequestParam boolean exception) {
  return embedService.saveAC(new ADocument("张三"), new CDocument("李四"), exception);
}

@Override
public Mono<Boolean> saveAC(ADocument aDocument, CDocument cDocument, boolean exception) {
  return reactiveMongoTemplate.inTransaction()
    //所有文档的持久化操作都只能在单独一个execute函数中汇总实现
    .execute(action -> Flux.fromIterable("1", "2", "3")
             //如果是个flux此处要用concatMap保持有序不能用flatMap
             .concatMap(i -> action.insert(aDocument)
                        .flatMap(a -> {
                          cDocument.setName(a.getName() + "copy");
                          return action.insert(cDocument)
                            .map(d -> {
                              if (exception) {
                                //测试跨文档的异常回滚
                                throw Exceptions.propagate(new RuntimeException("模拟异常的出现"));
                              }
                              return d;
                            });
                        }));
            )
    //如果里面返回的就是一个flux则不需要使用next
    //.next()
    .map(list -> {
      //需要注意，在execute之外的函数中产生的异常，不会触发事务的回滚。
      //                    if (exception) {
      //                        throw Exceptions.propagate(new RuntimeException("模拟异常的出现"));
      //                    }
      return Boolean.TRUE;
    });
}
```