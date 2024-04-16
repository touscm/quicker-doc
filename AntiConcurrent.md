# 防并发

分布式防止并发处理, 原理为通过Redis的原子操作实现. 使用方式有两种
 * 在Controller接口添加注解
 * 服务代码直接引用

## Controller注解

`@AntiConcurrent`注解用于防止Controller接口重复提交, `type`字段定义类型标识, `keys`字段定义请求参数为Map时的key的值, `timeout`字段定义锁定超时时间(秒), `message`字段定义错误消息, 接口请求参数为模型类时需要在模型类关键字段添加`AntiConcurrentKey`注解, 示例如下
```java
public class UserModel extends BaseModel {
    @AntiConcurrentKey
    private String id;
    private String username;
    private String password;
    private String status;
    private String createName;
    private String createBy;
    private Date createDate;
    private String updateName;
    private String updateBy;
    private Date updateDate;
}

@RestController
@RequestMapping("user")
public class AccountController {
    @Resource
    private UserDao userDao;

    @RequestMapping("test1")
    @AntiConcurrent(type = "anti-concurrent-key-1", timeout = 30, message = "重复提交, 请稍后重试")
    public RestResult<User> testUpdate1(@RequestBody UserModel model) {
        int updateCount = userDao.update(new Query<>(User::getId, model.getId()).set(User::setUpdateDate, new Date()));
        return new RestResult<User>().setStatus(RestResultStatus.Success).setMessage("" + updateCount);
    }

    @RequestMapping("test2")
    @AntiConcurrent(type = "anti-concurrent-key-2", keys = {"id"}, timeout = 30, message = "重复提交, 请稍后重试")
    public RestResult<User> testUpdate2(@RequestBody Map<String, String> model) {
        int updateCount = userDao.update(new Query<>(User::getId, model.get("id")).set(User::setUpdateDate, new Date()));
        return new RestResult<User>().setStatus(RestResultStatus.Success).setMessage("" + updateCount);
    }
}
```

## 服务调用

服务代码防止重复处理的逻辑是先检查是否重复处理, 检查通过后进行业务处理, 处理完成后释放重复检查锁. 示例如下

```java
    @Bean
    @Resource
    public CommandLineRunner testAntiConcurrent(IConcurrentService concurrentService) {
        return (args) -> {
            if(concurrentService.isConcurrent("anti-concurrent-type", "key-value", 30)) {
                System.out.println("重复处理");
                return;
            }

            // TODO 业务处理

            concurrentService.release("anti-concurrent-type", "key-value");
        };
    }
```
