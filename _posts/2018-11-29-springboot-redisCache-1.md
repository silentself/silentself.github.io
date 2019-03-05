---
layout: post
title:  springboot+redis缓存的使用<1>
excerpt: "使用redis缓存经常被命中的查询..."
categories: [java]
tags: [springboot]
comments: true
---

环境：springboot2.0.2

<!--more-->

#### 1、开启缓存

```java
@EnableCaching
```

在项目主启动类加上注解@EnableCaching开启缓存

#### 2、创建Redi缓存配置类，自定义缓存规则为json，默认是将对象直接序列化到redis中

```java
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
        throws UnknownHostException {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        Jackson2JsonRedisSerializer<Object> serializer = new Jackson2JsonRedisSerializer<>(Object.class);
        template.setDefaultSerializer(serializer);
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory)
        throws UnknownHostException {
        StringRedisTemplate template = new StringRedisTemplate();
        Jackson2JsonRedisSerializer<String> serializer = new Jackson2JsonRedisSerializer<>(String.class);
        template.setDefaultSerializer(serializer);
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean(name = "cacheManager")
    @Primary
    public CacheManager cacheManager(ObjectMapper objectMapper, RedisConnectionFactory redisConnectionFactory) {
        Jackson2JsonRedisSerializer<Object> serializer = new Jackson2JsonRedisSerializer<>(Object.class);
        RedisCacheConfiguration cacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
            .disableCachingNullValues()
            // .computePrefixWith(cacheName ->
            // "yourAppName".concat(":").concat(cacheName).concat(":"))
            .serializeKeysWith(
            RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(serializer));

        return RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(cacheConfiguration).build();
    }
}
```

#### 3、在service层设置缓存

```java
@Service
@CacheConfig(cacheNames = "dropDownbox")
public class DropDownboxImpl implements IDropDownbox {
    
    @Autowired
    DropDownboxDao dropDownboxDao;
    
    @Override
    @Cacheable(value = "getAllOrganizationList", key = "#root.caches[0]")
    public List<OrganizationForm> getAllOrganizationList() {
        return dropDownboxDao.selectOrgnameList();
    }
    @Override
    @Cacheable(value = "getMcnListByUuid", key = "#root.caches[0]")
    public List<MCNSimpleForm> getMcnListByUuid(Integer uuid) {
        return dropDownboxDao.selectMcnListByUuid(uuid);
    }
    @Override
    @Cacheable(value = "getAreaList", key = "#root.caches[0]")
    public List<AreaProvinceForm> getAreaList() {
        List<AreaProvinceForm> list = dropDownboxDao.selectAreaList();
        return list;
    }
}
```

##### 然而，在工作中使用时缺发现了问题：

```java
Cannot convert org.springframework.data.redis.cache.RedisCache to String. Register a Converter or override toString().
```

