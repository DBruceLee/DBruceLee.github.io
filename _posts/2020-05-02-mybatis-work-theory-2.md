---
layout: post
title:  mybatis 体系结构与工作原理（下）
categories: Mybatis
description:  mybatis 体系结构与工作原理（下）
keywords: mybatis、theory
---

### 体系结构（下）



#### 涉及到的GoF设计模式有哪些？

1. 装饰器
2. 模板方法
3. 代理
4. 拦截器



**SqlSessionFactoryBuilder().build(stream)**

<script src="/assets/js/mermaid.min.js"></script>


**sqlSessionFactory.openSession**

<div class="mermaid">
sequenceDiagram
    sqlSessionFactory->>sqlSessionFactory: openSession
    sqlSessionFactory->>DefaultSqlSessionFactory: openSession
    DefaultSqlSessionFactory->>DefaultSqlSessionFactory: openSessionFromDataSource
    DefaultSqlSessionFactory->>Configuration: getEnvironment()
    Configuration-->>DefaultSqlSessionFactory: return Environment
    DefaultSqlSessionFactory->>DefaultSqlSessionFactory: getTransactionFactoryFromEnvironment(environment)
    DefaultSqlSessionFactory->>DefaultSqlSessionFactory: transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit)
    DefaultSqlSessionFactory->>Configuration: newExecutor(tx, execType)
    alt is BATCH
    Configuration-->>DefaultSqlSessionFactory: return BatchExecutor
    else is REUSE
    Configuration-->>DefaultSqlSessionFactory: return ReuseExecutor
    else is SIMPLE
    Configuration-->>DefaultSqlSessionFactory: return SimpleExecutor
    end
    alt is cacheEnabled
    Configuration-->>DefaultSqlSessionFactory: return CachingExecutor
    else is not cacheEnabled
    Configuration-->>DefaultSqlSessionFactory: return SimpleExecutor
    end
    DefaultSqlSessionFactory->>sqlSessionFactory: return DefaultSqlSession
</div>

<div class="mermaid">
sequenceDiagram
    A->>B: Query
    B->>C: Forward query
    Note right of C: Thinking...
    C->>B: Response
    B->>A: Forward response
</div>