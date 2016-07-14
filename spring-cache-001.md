# spring cache 与 Redis 对接

常用缓存注解
=============
案例一：
-------------

``` java
@Cacheable(value="entity",key="'entity_'+#id",unless = "#result == null")
public T findById(String id)
```
* 描述：@Cacheable用于缓存返回对象，如果缓存中有对应的对象，将不会执行具体方法，直接返回缓存对象

* 参数：
      value:缓存名称    可根据缓存为单位设置过期时间(配置   cache.redis.expires 配置项)
      key:缓存的key值，支持spel表达式  #id表示传入参数id  本例中缓存key类似：entity_212341234
      unless:表示如果方法返回值为空，则不缓存，不缓存空值

案例二：
-------------
``` java
@Caching(
        evict = {
                @CacheEvict(value = "entity", key = "'entity_' + #entity.id", beforeInvocation = true)
        },
 put = {
                @CachePut(value = "entity", key = "'entity_' + #entity.id",unless = "#entity == null")
        }
)
public T save(T entity) {
```
- 描述：保存对象时，表示对象属性发生了修改，需要更新对应缓存的值，更新缓存的值需要有2个步骤，先删除缓存对应值，再将新的值放入缓存
- 参数：
      evict：清除动作  将执行@CacheEvict注解
      put：将新的值放入缓存，put动作将会在调用具体方法之前执行
      beforeInvocation：标识清除缓存动作在执行具体方法之前执行还是之后执行

案例三：
-------------
``` java
@CacheEvict(value = "entity", key = "'entity_' + #obj.id", beforeInvocation = true)
public void logicDelete(T obj) throws IllegalAccessException, InvocationTargetException{
```
描述：删除指定对象时清除缓存


如何设定缓存过期时间
=============

RedisCacheManager可以支持设置缓存过期时间的一个map集合

``` java
@ComponentScan
@EnableCaching
@Configurable
public class FrameworkCacheConfig extends CachingConfigurerSupport {

    public static final Logger logger = LoggerFactory.getLogger(FrameworkCacheConfig.class);
    @Bean
    public CacheManager cacheManager(RedisTemplate redisTemplate) {
        RedisCacheManager rcm = new RedisCacheManager(redisTemplate);
//        rcm.setExpires();

        Map<String,String> expires = SystemConfig.getMapProperty("cache.redis.expires");
        Map<String,Long> expiresLong = new HashedMap();
        for (String key:expires.keySet()) {
            Long expireValue = Long.parseLong(expires.get(key));
            if (logger.isDebugEnabled()){
                    logger.debug("设置缓存过期时间->{}:{}",key,expireValue);
             }
            expiresLong.put(key,expireValue);
        }
        rcm.setExpires(expiresLong);

        return rcm;
    }

```
