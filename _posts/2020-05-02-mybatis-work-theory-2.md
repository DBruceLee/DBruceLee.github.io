---
layout: post
title:  mybatis 体系结构与工作原理（下）
categories: Mybatis
description:  mybatis 体系结构与工作原理（下）
keywords: mybatis、theory
---

### mybatis 体系结构与工作原理（下）



#### 涉及到的GoF设计模式有哪些？

1. 装饰器
2. 模板方法
3. 代理
4. 拦截器

**SqlSessionFactoryBuilder().build(stream)**

```mermaid
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
```

**sqlSessionFactory.openSession**

```mermaid
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

```

**sqlSession.getMapper()**

```mermaid
sequenceDiagram
	businessCode->>sqlSession: getMapper()
	sqlSession->>Configuration: getMapper()
	Configuration->>MapperRegistry: getMapper()
	MapperRegistry->>MapperRegistry: knownMappers.get();
	Note over MapperRegistry: 获取mapperProxyFactory代理对象
	MapperRegistry->>MapperProxyFactory: newInstance(sqlSession)
	MapperProxyFactory->>MapperProxyFactory: newInstance(mapperProxy)
	MapperProxyFactory-->>MapperRegistry: return MapperProxy 对象
	MapperRegistry-->> Configuration: return MapperProxy 对象
	Configuration-->>sqlSession: return MapperProxy 对象
	sqlSession-->>businessCode: return MaperProxy 对象
```

**mapper.selectOne()**

```mermaid
sequenceDiagram
	businessCode->> mapperProxy: selectOne
	mapperProxy-> mapperProxy: invoke
	alt is getDeclaringClass
	mapperProxy->> mapperProxy: method.invoke()
	else is not
	mapperProxy->>MapperMethodInvoker: cachedInvoker()
	alt isDefault ?
	MapperMethodInvoker->>MapperMethodInvoker: DefaultMethodInvoker.invoke()
	else not default
	MapperMethodInvoker->>PlainMethodInvoker: invoke()
	PlainMethodInvoker->>MapperMethod: excute()
	alt is INSERT
	MapperMethod->>sqlSession: insert
	else is UPDATE
	MapperMethod->>sqlSession: update
	else is DELETE
	MapperMethod->>sqlSession: delete
	else is SELECT
	MapperMethod->>sqlSession: select
	sqlSession->>sqlSession: selectList
	sqlSession->>sqlSession: configuration.getMappedStatement
	sqlSession->>cacheExecutor: query
	alt open Cache?
	cacheExecutor->>cacheExecutor: TransactionalCacheManager.getObject
	alt cache have data?
	cacheExecutor-->>sqlSession: return cache data
	else no cache data
	Note over cacheExecutor: find data from DB
	cacheExecutor->>cacheExecutor: save data to cache
	end
	else no open cache
	sqlSession->>baseExecutor: query
	Note over baseExecutor: first find cache
	baseExecutor->>baseExecutor: queryFromDB
	baseExecutor->>baseExecutor: save data to cache
	end
	else is FLUSH
	MapperMethod->>sqlSession: flushStatements
	end
	end
	mapperProxy->>mapperProxy: invoke()
	end
```



