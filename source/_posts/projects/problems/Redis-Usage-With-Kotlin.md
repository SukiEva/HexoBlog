---
title: Spring Boot（Kotlin）使用Redis的踩坑记录
toc: true
categories: [Projects, Problems]
tags: 
	- Redis
	- Spring Boot
	- Kotlin
date: 2022-07-17 20:50
updated: 2022-07-17 22:15
references:
	- title: Kotlin之在Gradle中无参（no-arg）编译器插件的使用
	  url: https://segmentfault.com/a/1190000020958806
	- title: "Spring-Data-Redis--解决java.lang.ClassCastException: java.util.LinkedHashMap cannot be cast to xxx"
	  url: "https://www.voycn.com/article/spring-data-redis-jiejuejavalangclasscastexception-javautillinkedhashmap-cannot-be-cast-xxx"
	- title: Spring Boot demo系列（十）：Redis缓存
	  url: https://zhuanlan.zhihu.com/p/352574006  
---

虽然理论上 Koltin “NTR” 了 Java 的一切，但是在实际开发中总能碰到许多奇奇怪怪的问题。

<!-- more -->

## 添加依赖

首先是 Redis 的依赖：

```kotlin
dependencies {
	implementation("org.springframework.boot:spring-boot-starter-data-redis")
}
```

这个包会通过反射来加载一些 class，这就要求 class 必需有一个无参的构造函数，而 Kotlin 的 data class 默认没有无参构造函数，并且 data class 默认为 final 类型，不可以被继承，会出现下列错误：

`org.springframework.data.redis.serializer.SerializationException: Could not read JSON: Cannot construct instance of …`

Kotlin 官方为我们提供了两个插件：

- 无参编译器插件 NoArg 插件的作用是：为指定的类在编译时添加一个无参的构造。
- 全开放编译器插件 AllOpen 的作用是：使用指定的注解标注类而其成员无需显式使用 open 关键字打开。

**Spring Boot 只需要 NoArg 插件，因为添加的 kotlin-spring 插件包含 AllOpen 了，会自动给 Spring 注解类 open**，下面的只作为演示：

```kotlin
plugins {
    val kotlinVersion = "1.6.21"
    id("org.jetbrains.kotlin.plugin.noarg") version kotlinVersion
    id("org.jetbrains.kotlin.plugin.allopen") version kotlinVersion
    kotlin("plugin.jpa") version kotlinVersion
}

allOpen {
    annotation("com.gitfy.gitfyapi.util.AllOpen")
}

noArg {
    annotation("com.gitfy.gitfyapi.util.NoArg")
}
```

特别说明一下这俩插件的使用，除了在 `plugins` 里添加依赖，还需要加上上面的俩注解声明，其中前面的是声明的注解类所在包，比如我在 `om.gitfy.gitfyapi.util` 包下新建 `Annotations.kt`：

```kotlin
package com.gitfy.gitfyapi.util

annotation class NoArg

annotation class AllOpen
```

这些弄完就可以在 data class 里使用注解了：

```kotlin
@NoArg
data class Repo(
    var platform: String,
    var owner: String,
    var repo: String,
)
```

> 如果项目中已经添加了 `kotlin-jpa` 插件，那么基本上就不必单独添加无参插件了。
>
> `kotlin-jpa` 对无参插件做了包装，当使用 `@Entity`、` @Embeddable` 与 ` @MappedSuperclass` 这几个注解时，都会默认支持无参注解的。
>
> 当然也可以手动给 data class 赋初始值。

## 添加配置

这里将 `key` 序列化为 `string`，`value` 序列化为 `json`。

SpringBoot 的缓存使用 jackson 来做数据的序列化与反序列化，如果默认使用 Any 作为序列化与反序列化的类型，则其只能识别基本类型，遇到复杂类型时，jackson 就会先序列化成 LinkedHashMap ，然后再尝试强转为所需类别，这样大部分情况下会强转失败，出现以下错误：

`class java.util.LinkedHashMap cannot be cast to class …`

解决方法是修改 RedisTemplate 这个 bean 的 valueSerializer，即设置默认类型。

```kotlin
@Configuration
class RedisConfig {

    /**
     * 配置Jackson2JsonRedisSerializer序列化策略
     *
     * @return Jackson2JsonRedisSerializer<Any>
     */
    private fun serializer(): Jackson2JsonRedisSerializer<Any> {
        // 使用 Jackson2JsonRedisSerializer 来序列化和反序列化 redis 的 value 值
        val jackson2JsonRedisSerializer = Jackson2JsonRedisSerializer(Any::class.java)
        val objectMapper = ObjectMapper().apply {
            // 指定要序列化的域，field,get和set,以及修饰符范围，ANY是都有包括private和public
            setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY)
            // 指定序列化输入的类型，类必须是非final修饰的，final修饰的类，比如String,Integer等会跑出异常
           // 这个必须加！！！不然 class java.util.LinkedHashMap cannot be cast to class XXX
            activateDefaultTyping(
                this.polymorphicTypeValidator,
                ObjectMapper.DefaultTyping.NON_FINAL,
                JsonTypeInfo.As.PROPERTY
            )
        }
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper)
        return jackson2JsonRedisSerializer
    }

    /**
     * Redis template 配置
     *
     * @param redisConnectionFactory redis连接工厂
     * @return RedisTemplate
     */
    @Bean
    fun redisTemplate(redisConnectionFactory: RedisConnectionFactory): RedisTemplate<String, Any> {
        val redisTemplate = RedisTemplate<String, Any>()
        val stringRedisSerializer = StringRedisSerializer()
        val jackson2JsonRedisSerializer = serializer()
        redisTemplate.apply {
            // 配置连接工厂
            setConnectionFactory(redisConnectionFactory)
            // 配置 key 序列化方式
            keySerializer = stringRedisSerializer
            // 配置 value 序列化方式: 使用 Jackson2JsonRedisSerializer
            valueSerializer = jackson2JsonRedisSerializer
            // 配置 hash key 序列化方式
            hashKeySerializer = stringRedisSerializer
            // 配置 hash value 序列化方式
            hashValueSerializer = jackson2JsonRedisSerializer
            afterPropertiesSet()
        }
        return redisTemplate
    }

}
```

## 封装工具

为了方便，封装一个简单的工具类：

```kotlin
@Component
class RedisUtil {

    @Autowired
    private lateinit var redisTemplate: RedisTemplate<String, Any>

    /**
     * 读取数据
     *
     * @param key
     * @return Any?
     */
    fun get(key: String): Any? {
        if (key == "") return null
        try {
            return redisTemplate.opsForValue().get(key)
        } catch (e: Exception) {
            e.printStackTrace()
        }
        return null
    }

    /**
     * 写入数据
     *
     * @param key
     * @param value
     * @param expireTime
     * @return Boolean
     */
    fun set(key: String, value: Any, expireTime: Long? = null): Boolean {
        if (key == "") return false
        try {
            redisTemplate.opsForValue().set(key, value)
            expireTime?.let {
                redisTemplate.expire(key, it, TimeUnit.SECONDS)
            }
            return true
        } catch (e: Exception) {
            e.printStackTrace()
        }
        return false
    }
}
```

## 测试使用

```kotlin
@RunWith(SpringJUnit4ClassRunner::class)
@SpringBootTest
class RedisUtilTest {

    @Autowired
    private lateinit var redisUtil: RedisUtil

    @Test
    fun get() {
        val repo = redisUtil.get("github:SukiEva:GitfyApi")
        println(repo)
        val obj:Repo = repo as Repo
        println(obj)
    }

    @Test
    fun set() {
        val repo = Repo("github", "SukiEva", "GitfyApi")
        redisUtil.set("github:SukiEva:GitfyApi", repo)
    }
}
```

如果你也是 Kotlin 玩家，这里肯定会碰到报错：

`org.springframework.data.redis.serializer.SerializationException: Could not read JSON: Could not resolve subtype of [simple type, class java.lang.Object]: missing type id property '@class'`

在 `Java` 中，实体类没有任何额外配置，`Redis` 序列化/反序列化一样没有问题，是因为值序列化器 `GenericJackson2JsonRedisSerializer`，该类会自动添加一个 `@class` 字段，因此不会出现上面的问题。

但是在 `Kotlin` 中，类默认不是 `open` 的，也就是无法添加 `@class` 字段，因此便会反序列化失败。

首先当然可以通过不用 data class 来解决，直接给普通 class open，这当然不是我们想要的，所以需要添加一个类注解 `@JsonTypeInfo`：

```kotlin
@NoArg
@JsonTypeInfo(use = JsonTypeInfo.Id.CLASS)
data class Repo(
    var platform: String,
    var owner: String,
    var repo: String,
)
```

该注解的 `use` 用于指定类型标识码，该值只能为 `JsonTypeInfo.Id.CLASS`。

这时 Redis 中的结构是：

```json
{
    "@class": "com.gitfy.gitfyapi.pojo.Repo",
    "platform": "github",
    "owner": "SukiEva",
    "repo": "GitfyApi"
}
```

并且此时 `get` 方法也能正常获取值，并强制类型转换为对应的对象。
