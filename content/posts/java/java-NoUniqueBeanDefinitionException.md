---
title: "Java NoUniqueBeanDefinitionException"
date: 2023-09-18T17:01:30+08:00
draft: false
tags: ["Java"]
categories: ["Java"]
---

在使用Spring Web + Mybatis-plus + MybatisX插件，创建Java工程后，启动时总是报不唯一的bean，反复检查了几遍，也没有找到问题。

```
Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'com.coding.generator.service.ApiNotesService' available: expected single matching bean but found 2: apiNotesServiceImpl,apiNotesService
```

经过搜索，找到了2种解决方法

1. 按名字匹配注入
    ```
    @Resource(name = "apiNotesServiceImpl")
    private ApiNotesService notesService;
    ```
    或者
    ```
    @Autowired
    @Qualifier("apiNotesServiceImpl")
    private ApiNotesService notesService;
    ```
2. 提供准确的`@MapperScan`路径
   ```
   @MapperScan("com.coding.generator.mapper")
   ```