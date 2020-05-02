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
<div class="mermaid">
sequenceDiagram
  SqlSessionFactory->>SqlSessionFactoryBuilder: bulid
  SqlSessionFactoryBuilder->>XMLConfigBuilder: bulid
  XMLConfigBuilder->>XMLConfigBuilder: parse()
  XMLConfigBuilder->>Configuration: parseConfiguration(XNode root)
  Configuration->>Configuration: setVariables()
  Configuration-->>XMLConfigBuilder: return
  XMLConfigBuilder->>Configuration: settingsAsProperties("settings")
  Configuration->>Configuration: metaConfig.hasSetter()
  Configuration-->>XMLConfigBuilder: return Properties
  XMLConfigBuilder->>XMLConfigBuilder: loadCustomVfs(settings)
  XMLConfigBuilder->>XMLConfigBuilder: loadCustomLogImpl(settings)
  XMLConfigBuilder->>XMLConfigBuilder: typeAliasesElement("typeAliases")
  XMLConfigBuilder->>TypeAliasRegistry: registerAlias() 
  TypeAliasRegistry-->>XMLConfigBuilder: return
  XMLConfigBuilder->>XMLConfigBuilder: pluginElement("plugins")
  XMLConfigBuilder->>XMLConfigBuilder: objectFactoryElement("objectFactory")
  XMLConfigBuilder->>XMLConfigBuilder: objectWrapperFactoryElement("objectWrapperFactory")
  XMLConfigBuilder->>XMLConfigBuilder: reflectorFactoryElement("reflectorFactory")
  XMLConfigBuilder->>XMLConfigBuilder: settingsElement(settings)
  XMLConfigBuilder->>Configuration: environmentsElement("environments")
  Configuration->>Configuration: configuration.setEnvironment()
  Configuration-->>XMLConfigBuilder: return
 XMLConfigBuilder->>XMLConfigBuilder: databaseIdProviderElement("databaseIdProvider")
 XMLConfigBuilder->>XMLConfigBuilder: typeHandlerElement("typeHandlers")
 alt is package
 XMLConfigBuilder->>XMLConfigBuilder: typeHandlerRegistry.register(typeHandlerPackage)
 else is class
 XMLConfigBuilder->>XMLConfigBuilder: typeHandlerRegistry.register(typeHandlerClass)
 end
 XMLConfigBuilder->>XMLConfigBuilder: mapperElement(XNode parent)
 alt is package
 XMLConfigBuilder->>XMLConfigBuilder: configuration.addMappers(mapperPackage)
 else is resource
 XMLConfigBuilder->>XMLConfigBuilder: XMLMapperBuilder(resource) 
 else is url
 XMLConfigBuilder->>XMLConfigBuilder: XMLMapperBuilder(url) 
 else is class
 XMLConfigBuilder->>XMLConfigBuilder: configuration.addMapper(mapperInterface)
 end
 XMLConfigBuilder-->>SqlSessionFactoryBuilder: return Configuration
 SqlSessionFactoryBuilder->>SqlSessionFactoryBuilder: build(Configuration config)
 SqlSessionFactoryBuilder-->>SqlSessionFactory: return DefaultSqlSessionFactory
</div>

**sqlSessionFactory.openSession**
<script src="/assets/js/mermaid.min.js"></script>
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

