# 途悠e站

## 代码使用说明

### 避免重复提交注解使用

#### 直接调用
```java
    @Autowired
    ConcurrentService concurrentService;

public enum ConcurrentType { Order }

    // 类型
    ConcurrentType type = ConcurrentType.Order;
    // 键值
    String key = "123";
    // 超时(秒)
    int timeout = 0;

// 重复提交检查
    if (concurrentService.isConcurrent(type, key, timeout)) {
            // TODO 重复提交处理
            }

            // 释放检查标记缓存
            concurrentService.release(type, key);
```

#### 注解使用
>注解使用有作用域概念，分全局作用域和会话作用域，默认为会话作用域。<br/>
>会话作用域标识缓存在请求完成时或者超时时释放<br/>
>全局作用域标识缓存只在超时时释放<br/>
请求参数为模型类时示例
```java
public class User {
    private String id;

    @AntiConcurrentKey
    private String username;
}

@RestController
public class UserController {
    @AntiConcurrent(ConcurrentCategory.AddUser)
    @AuthIdentity
    @RequestMapping("/driver/user/insert")
    public Result insert(@RequestBody User model) {
        // TODO
    }
}
```

请求参数为Map时示例
```java
@RestController
public class UserController {
    @AntiConcurrent(scope = ConcurrentScope.Global, type = "EditUser", , keys = {"id"}, timeout = 5, message = "重复提交")
    @AuthIdentity
    @RequestMapping("/driver/user/insert")
    public Result edit(@RequestBody Map<String, String> model) {
        // TODO
    }
}
```

### Quicker Mybatis 使用

#### 定义实体类
```java
import com.touscm.quicker.domain.BaseEntry;
import javax.persistence.Column;
import javax.persistence.Id;

public class MyEntity extends BaseEntry {
    @Id
    private String id;
    @Column(name = "name", length = 255)
    private String name;
    private long value;
    
    public String getId() { return id; }
    public void setId(String id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public long getValue() { return value; }
    public void setValue(long value) { this.value = value; }
}
```

#### 定义DAO
>注意, dao类不能包含名为count, sumValue, sum, querySingle, queryList, queryPaging, insert, update, delete的方法
```java
import com.touscm.quicker.data.mybatis.IDao;
import javax.persistence.Table;

@Table(name = "my_table_name")
public interface MyDao extends IDao<MyEntity> {
}
```

#### 查询示例
```java
import com.touscm.quicker.data.Query;
import com.touscm.quicker.domain.web.PagingEntry;
import com.touscm.quicker.domain.web.ReqCondition;

public class MyClass {
    @Resource MyDao dao;
    
    public void testQuerySingle() {
        Optional<MyEntity> result;

        result = dao.querySingle(new Query<User>().where(MyEntity::getId, "id-value").and(MyEntity::getName, "name-value"));
        System.out.println(EntryUtils.toString(result));

        result = dao.querySingle(Arrays.asList(new ReqCondition("id", "id-value"), new ReqCondition("name", "name-value")));
        System.out.println(EntryUtils.toString(result));
    }

    public void testQueryList() {
        List<MyEntity> result;

        result = dao.queryList(new Query<MyEntity>().where(MyEntity::getId, "id-value").and(MyEntity::getName, "name-value"));
        System.out.println(EntryUtils.toString(result));

        result = dao.queryList(Arrays.asList(new ReqCondition("id", "id-value"), new ReqCondition("name", "name-value")));
        System.out.println(EntryUtils.toString(result));
    }

    public void testQueryPaging() {
        PagingEntry<MyEntity> result;
        int page = 1, pageSize = 10;

        result = dao.queryPaging(new Query<MyEntity>().where(MyEntity::getId, "id-value").and(MyEntity::getName, "name-value"), page, pageSize);
        System.out.println(EntryUtils.toString(result));

        result = dao.queryPaging(Arrays.asList(new ReqCondition("id", "id-value"), new ReqCondition("name", "name-value")), page, pageSize);
        System.out.println(EntryUtils.toString(result));
    }
}
```

#### 统计示例
```java
public class MyClass {
    @Resource MyDao dao;

    public void testCount() {
        long count;

        count = dao.count(new Query<MyEntity>().where(MyEntity::getId, "id-value").and(MyEntity::getName, "name-value"));
        System.out.println(count);

        count = dao.count(Arrays.asList(new ReqCondition("id", "id-value"), new ReqCondition("name", "name-value")));
        System.out.println(count);
    }

    public void testSumValue() {
        long sum = dao.sumValue(new Query<MyEntity>().where(MyEntity::getId, "id-value").and(MyEntity::getName, "name-value"), User::getValue);
        System.out.println(sum);
    }

    public void testSum() {
        Optional<MyEntity> sum = dao.sum(new Query<MyEntity>().sumSelect(MyEntity::getValue).where(MyEntity::getName, "name-value"), MyEntity.class);
        System.out.println(sum);
    }
}
```

#### 添加示例
```java
public class MyClass{
    @Resource MyDao dao;

    public void testInsert(UserDao dao) {
        int insertCount;

        insertCount = dao.insert(new Query<MyEntity>().set(MyEntity::setId, "id-value").set(MyEntity::setName, "name-value"));
        System.out.println(insertCount);

        insertCount = dao.insert(new MyEntity().setId("id-value").setName("name-value"));
        System.out.println(insertCount);
    }
}
```

#### 更新示例
```java
public class MyClass{
    @Resource MyDao dao;

    public void testUpdate() {
        int updateCount;

        updateCount = dao.insert(new Query<MyEntity>().where(MyEntity::getId, "id-value").set(MyEntity::setName, "name-value"));
        System.out.println(updateCount);

        updateCount = dao.insert(new MyEntity().setId("id-value").setName("name-value"));
        System.out.println(updateCount);
    }
}
```
