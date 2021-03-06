




## 3. Spring的@Transactional事务失效

### 3.1.自身调用问题（重要）

```java
// 一个类中调用会让事务失效（没有代理）
// 正确是方式是：
//   方式1.类中注入自己，用注入的对象再调用另外一个方法
//   方式2.将调用的方法抽出到另一个service中
@Service
public class OrderServiceImpl implements OrderService {
    public void update(Order order) {
        updateOrder(order);
    }
    @Transactional
    public void updateOrder(Order order) {
        // update order
    }
}
```

```java
// 一个类中调用会让事务失效（没有代理）
// 正确是方式是：
//   方式1.类中注入自己，用注入的对象再调用另外一个方法
//   方式2.将调用的方法抽出到另一个service中
@Service
public class OrderServiceImpl implements OrderService {
    @Transactional
    public void update(Order order) {
        updateOrder(order);
    }
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void updateOrder(Order order) {
        // update order
    }
}
```


### 3.2 异常被处理了（重要）

```java
// 异常被捕获不能抛出，事务不能回滚
@Service
public class OrderServiceImpl implements OrderService {
    @Transactional
    public void updateOrder(Order order) {
        try {
            // update order
        } catch {

        }
    }
}
```

### 3.3 异常类型错误（重要）

```java
// 默认回滚的是异常是RuntimeException。
// 如果想让事务生效，需要：@Transactional(rollbackFor = Exception.class)
@Service
public class OrderServiceImpl implements OrderService {
    @Transactional
    public void updateOrder(Order order) {
        try {
            // update order
        } catch {
            throw new Exception("更新错误");
        }
    }
}
```



### 3.4 数据库引擎不支持事务

MyISAM 引擎是不支持事务操作


### 3.5 没有被 Spring 管理

```java
// @Service
public class OrderServiceImpl implements OrderService {
    @Transactional
    public void updateOrder(Order order) {
        // update order
    }
}
```


### 3.6 方法不是public的

`@Transactional` 只能用于 public 的方法上。如果要用在非 public 方法上，可以开启 `AspectJ` 代理模式



### 3.7 数据源没有配置事务管理器

@EnableTransactionManagement


### 3.8 不支持事务 

@Transactional(propagation = Propagation.NOT_SUPPORTED)



### 3.9 方法用final或者static修饰



### 3.10 多线程调用

```java
/**
* spring的事务是通过数据库连接来实现的。
* 同一个事务，其实是指同一个数据库连接，只有拥有同一个数据库连接才能同时提交和回滚。
* 不同线程连接的数据库连接也不一样，所以是不同的事务
*/

/**
* doOtherThing方法中抛了异常，add方法也回滚是不可能的
*/
@Slf4j
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;
    @Autowired
    private RoleService roleService;

    @Transactional
    public void add(UserModel userModel) throws Exception {
        userMapper.insertUser(userModel);
        new Thread(() -> {
            roleService.doOtherThing();
        }).start();
    }
}
```

```java
@Service
public class RoleService {
    @Transactional
    public void doOtherThing() {
        System.out.println("保存role表数据");
    }
}
```