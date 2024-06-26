# 入门

入门文档以最简项目结构开始, 目的是对框架有一个比较清晰的认识, 关于详细内容请参照功能文档.

## 代码结构
目录结构为:<br/>

```
project
└───auto-insurance-domain
│   └───src
│       pom.xml
└───auto-insurance-service
│   └───src
│       pom.xml
└───auto-insurance-web
│   └───src
│       pom.xml
│   pom.xml
```
`auto-insurance-domain`包括实体, 模型, 数据操作的代码实现;<br/>
`auto-insurance-service`包括业务代码实现;<br/>
`auto-insurance-web`包括 Web-API 的代码实现

## 编写代码
需要实现的代码如下

#### `auto-insurance-domain` 模块 - 实体类
```java
@Entity(vehicleId = "goods_template")
public class GoodsTemplate extends BaseEntity {
    @Id
    @GeneratedValue(generator = "guid")
    @GenericGenerator(vehicleId = "guid", strategy = "guid")
    private String id;
    private String vehicleId;
    private Date createTime;
    private Date updateTime;

    public GoodsTemplate() {
    }
    public GoodsTemplate(String vehicleId) {
        this.vehicleId = vehicleId;
        this.createTime = new Date();
    }

    public String getId() {
        return this.id;
    }
    public void setId(String value) {
        this.id = value;
    }

    public String getName() {
        return this.vehicleId;
    }
    public void setName(String value) {
        this.vehicleId = value;
    }

    public Date getCreateTime() {
        return createTime;
    }
    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }

    public Date getUpdateTime() {
        return updateTime;
    }
    public void setUpdateTime(Date updateTime) {
        this.updateTime = updateTime;
    }
}
```
实体类需继承自 `BaseEntity` 类<br/>
实体类需添加 `@Entity` 注解, 驼峰形式的实体类名会自动转换为下划线表名.<br/>
主键字段需添加 `@Id` 注解, `@GeneratedValue` 注解 和 `@GenericGenerator` 用于指定自动填值类型.

#### `auto-insurance-domain` 模块 - 模型类
```java
public class GoodsTemplateModel extends BaseModel {
    @Validation(policy = "edit")
    private String id;
    @Validation(policy = "add")
    private String vehicleId;
    private Date createTime;
    private Date updateTime;
    public String getId() {
        return this.id;
    }

    public void setId(String value) {
        this.id = value;
    }

    public String getName() {
        return this.vehicleId;
    }

    public void setName(String value) {
        this.vehicleId = value;
    }

    public Date getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }

    public Date getUpdateTime() {
        return updateTime;
    }

    public void setUpdateTime(Date updateTime) {
        this.updateTime = updateTime;
    }
}
```
模型类需继承自 `BaseModel` 类<br/>
注解 `@Validation` 用于的指定数据验证, `policy` 表示验证的策略, 如果该模型只有一种验证验证方式则 `policy` 无需指定, 当然在数据验证的时候也不能指定策略.例如:<br/>
定义
```java
    @Validation
    private String id;
```
对应
```java
    model.validate(Policy.Add.getName());
```
定义
```java
    @Validation(policy = "add")
    private String id;
```
对应
```java
    model.validate("add");
```

#### `carry-trucking-domain` 模块 - 数据操作类
```java
@Repository
public interface UserDao extends IDao<User> {
}
```
数据操作类需继承自 `IDao`, 其泛型参数为所要操作的实体类;
 数据操作默认实现方法如下:
 ```java
 public interface IBaseDao<TEntity extends BaseEntry> {
     long count(@NotNull Query<TEntity> var1);
     long count(@NotEmpty List<ReqCondition> var1);
     <V extends Number> V sumValue(@NotNull Query<TEntity> var1, @NotNull Getter<TEntity, V> var2);
     <R> Optional<R> sum(@NotNull Query<TEntity> var1, @NotNull Class<R> var2);
     <R> Optional<R> sum(@NotEmpty List<ReqCondition> var1, @NotNull Class<R> var2);
     Optional<TEntity> querySingle(@NotNull Query<TEntity> var1);
     Optional<TEntity> querySingle(@NotEmpty List<ReqCondition> var1);
     List<TEntity> queryList(@NotNull Query<TEntity> var1);
     List<TEntity> queryList(@NotEmpty List<ReqCondition> var1);
     PagingEntry<TEntity> queryPaging(@NotNull Query<TEntity> var1, int var2, int var3);
     PagingEntry<TEntity> queryPaging(@NotEmpty List<ReqCondition> var1, int var2, int var3);
     int insert(@NotNull Query<TEntity> var1, @NotNull Consumer<String> var2);
     int insert(@NotNull TEntity var1, @NotNull Consumer<String> var2);
     int update(@NotNull Query<TEntity> var1);
     int delete(@NotNull Query<TEntity> var1);
 }
 ```
IDao默认方法请求参数为`List<ReqCondition>`和`Query<TEntity>`两种类型, 使用示例如下<br>
 `List<ReqCondition>`:<br>
 ```java
 long count = userDao.count(Arrays.asList(new ReqCondition("name", "123"), new ReqCondition("phone", "13000000000")));
```
 `Query<TEntity>`:<br>
```java
 long count = userDao.count(new Query<User>(User::getName, "123").and(User::getPhone, "13000000000"));
```

#### `carry-trucking-service`模块 - 服务类
```java
 @Service
 public interface UserService extends IService<User, UserModel, UserViewModel, UserDao> {
 }
```
服务类需继承自 `IService` 接口, 其泛型参数依次为: 实体类, 模型类, 试图模型类, 数据操作类, 模型类和试图模型类可相同
`IService`默认实现了添加,查询,更新,删除等操作, 类似`IDao`默认实现

#### `carry-trucking-web` 模块 - 控制器
```java
@RestController
public class GoodsTemplateController extends BaseController<GoodsTemplate, GoodsTemplateModel, GoodsTemplateModel, GoodsTemplateServiceImpl> {

}
```
控制器需继承自 `BaseController` 类, 其泛型参数依次为: 实体类, 模型类, 视图模型类, 服务类.<br/>
控制器需添加 `@Controller` 注解, `@RestController` 表示该控制器返回 JSON 对象.<br/>
`BaseController`默认实现 `add`, `edit`, `deleteStatus`, `get`, `pagging` 方法.<br/>
`add` 方法, 路由为 "Controller/Add", 其接收 JSON 数据格式为:
```json
{
	"model": {
		"vehicleId": "货源模板 2018-08-01 18:54"
	}
}
```
其中 `model` 对应 `Controller` 定义的模型类.<br/>
`edit` 方法, 路由为 "Controller/Edit", 其接收 JSON 数据格式为:
```json
{
	"model": {
		"id": "xxx",
		"vehicleId": "货源模板 2018-08-01 20:54"
	}
}
```
其中 `model` 对应 `Controller` 定义的模型类.<br/>
`deleteStatus` 方法, 路由为 "Controller/Delete", 其接收 JSON 数据格式为:
```json
{
	"conditions": [
		{ "type": "GoodsTemplateId", "value": "48780417-957c-11e8-af7d-f0def19cb946" }
	]
}
```
其中 `conditions` -> `type` 的值为 实体类类名+实体类主键字段名, `conditions` -> `value` 的值为 主键的值.<br/>
删除操作可包含多个参数<br/>
<label style='color:red'>注意</label>:实体类需包含 `isDelete` 字段删除操作才能正常工作.<br/>
`get` 方法, 路由为 "Controller/Get", 其接收 JSON 数据格式为:
```json
{
	"conditions": [
		{ "type": "GoodsTemplateId", "value": "48780417-957c-11e8-af7d-f0def19cb946" }
	]
}
```
格式同删除操作
`pagging` 方法, 路由为 "Controller/Pagging", 其接收 JSON 数据格式为:
```json
{
	"page": 1,
	"pageSize": 20,
	"conditions": [
		{ "type": "Name", "value": "货源模板" }
	]
}
```

所有方法返回 JSON 格式为:
```json
{
	"status": 200, "message": "处理成功", "data": null
}
```
`status` 值可参照 ./carry-common/src/main/java/com/tyy/carry/common/web/RestResultStatus.java

#### Web Api 使用说明
除了/auth/login和/auth/forgetpassword之外的接口都需要在http header添加key为"Authorization", value为"Bearer eyJhbGciOiJOb25lIiwidHlwIjoiVG9rZW4ifQ.eyJtb2JpbGUiOiIiLCJuYW1lIjoidXNlcjEiLCJ0b2tlbiI6ImYyNjM4YjljM2U4NjRiN2Q5MDJhNjE2NmU4ZjZlNWE3In0."的内容;
其中value的值中Bearer + 空格后的内容根据用户登陆返回的信息通过Base64Url编码得到, 下面说明编码具体实现.
编码由header,payload,signature共3部分组成,中间同过"."符号合并.
`header`内容为
```json
{
    "typ": "Token",
    "alg": "None"
}
```
经过Base64编码后得到eyJhbGciOiJOb25lIiwidHlwIjoiVG9rZW4ifQ
`payload`内容为
```json
{
    "name": "用户名",
    "mobile": "手机号",
    "token": "登陆接口返回的sessionToken"
}
```
`signature`暂时不支持

示例
header:
```json
{
    "typ": "Token",
    "alg": "None"
}
```

payload:
```json
{
    "name": "username1",
    "mobile": "",
    "token": "458DDF038E7946A393ECA59771CB02FB"
}
```
编码后的结果为
Bearer eyJ0eXAiOiJUb2tlbiIsImFsZyI6Ik5vbmUifQ.eyJuYW1lIjoidXNlcm5hbWUxIiwidG9rZW4iOiI0NThEREYwMzhFNzk0NkEzOTNFQ0E1OTc3MUNCMDJGQiJ9.

#### 登陆接口变更
添加 "platformId" 参数, 该字段为平台ID;
字段 "loginType" 增加值; 明细如下:
0: 货主用户名登陆; 1: 车主手机号登陆; 2: 司机手机号登陆, 3: 车牌号登陆; 10: 系统用户登录, 使用用户名; 11: 平台用户登录, 使用用户名;

#### 认证方式变更
添加 "region", "role", "identity" 字段, 删除 "name", "mobile" 字段;
"region" 表示平台类型, 货主,车主,司机,平台管理员是该字段的值为 "Platform", 系统管理员时该字段值为 "System"
"role" 表示角色类型, 货主时该字段为 "Shipper", 车主时该字段值为 "Carowner", 车主登陆时该字段值为 "Driver", 车牌号登陆时该字段为 "CarNo"
"identity" 表示登陆识别字段, 货主登陆时表示用户名, 车主和司机登陆时表示手机, 车牌号登陆时表示为车牌号

#### 查询条件使用
查询, 更新, 删除处理条件判断, 默认使用 ```List<ReqCondition>```, 暂时只支持AND, OR未实现<br>
```ReqCondition```类定义为:
```java
public class ReqCondition {
    String type;
    String value;
}
```
type用于指定字段名, value用于指定数值<br>
##### 等于查询
```Arrays.asList(new ReqCondition("name", "name1"))```, 查询字段name等于name1的数据
##### 不等于查询
```Arrays.asList(new ReqCondition("neq_name", "name1"))```, 查询字段name不等于name1的数据
##### 大于查询
```Arrays.asList(new ReqCondition("gt_val", "1"))```, 查询字段val大于1的数据
##### 小于查询
```Arrays.asList(new ReqCondition("lt_val", "1"))```, 查询字段val小于1的数据
##### LIKE查询
```Arrays.asList(new ReqCondition("like_name", "a"))```, 查询字段name包含a的数据
##### IN查询
```Arrays.asList(new ReqCondition("in_val", "1"), new ReqCondition("in_val", "2"))```, 查询字段val是1或2的数据
##### ISNULL查询
```Arrays.asList(new ReqCondition("null_val", ""))```, 查询字段val是null的数据, 无需设值value字段
##### NOT ISNULL查询
```Arrays.asList(new ReqCondition("notnull_val", ""))```, 查询字段val不是null的数据, 无需设值value字段
##### Like查询
```Arrays.asList(new ReqCondition("start_val", "start"))```, 查询字段val是否以start开始
##### Like查询
```Arrays.asList(new ReqCondition("end_val", "end"))```, 查询字段val是否以end结尾
##### 为空查询
```Arrays.asList(new ReqCondition("empty_val", ""))```, 查询字段val是否为空字符串
##### 不为空查询
```Arrays.asList(new ReqCondition("notempty_val", ""))```, 查询字段val是否不为空字符串

#### 使用事务
使用范围为 <b>BaseService</b> 的实现类, 方法定义:<br>
```java
protected <T> Optional<ProcessResult<T>> doTransaction(Supplier<ProcessResult<T>> supplier)
```
参数为 <b>ProcessResult<T></b> 类型提供委托, <b>ProcessResult<T></b> 定义为<br>
```java
public class ProcessResult<T> {
    private boolean success;
    private T data;
}
```
<b>ProcessResult<T></b> 字段 success 为 ture 时执行处理提交, 否则处理回滚, 执行体已包含异常处理<br>
使用示例:<br>
```java
    public boolean active(List<String> ids, boolean usePermission) {
        Objects.requireNonNull(ids);

        try {
            List<TEntity> entities = this.getEntityListByIds(ids, usePermission);

            if (!CollectionUtils.isEmpty(entities)) {
                entities.forEach(a -> {
                    EntryUtils.setEntityDelete(a, false);
                    getSession().ifPresent(i -> a.setUpdateBy(i.getName()));
                    a.setUpdateTime(new Date());
                });

                AtomicBoolean ref = new AtomicBoolean();
                this.doTransaction(() -> new ProcessResult<>(!CollectionUtils.isEmpty(this.dao.saveAll(entities)))).ifPresent(a -> ref.set(a.isSuccess()));

                return ref.get();
            }
        } catch (Exception e) {
            logger.error(String.format("数据启用失败, Param:{ ids:%s, usePermission:%s }", EntryUtils.getEntryInfo(ids), usePermission), e);
        }

        return false;
    }
```

## 签名验签

### 系统签名
系统发送消息或者应答平台请求时会做签名处理, HTTP头报文如下:
```
TyyOpen-Timestamp: 1627374437
TyyOpen-Nonce: 150967fe0671410ba3a7525e5f5e2506
TyyOpen-Signature: CtcbzwtQjN8rnOXItEBJ5aQFSnIXESeV28Pr2YEmf9wsDQ8Nx25ytW6FXBCAFdrr0mgqngX3AD9gNzjnNHzSGTPBSsaEkIfhPF4b8YRRTpny88tNLyprXA0GU5ID3DkZHpjFkX1hAp/D0fva2GKjGRLtvYbtUk/OLYqFuzbjt3yOBzJSKQqJsvbXILffgAmX4pKql+Ln+6UPvSCeKwznvtPaEx+9nMBmKu7Wpbqm/+2ksc0XwjD+xlvlECkCxfD/OJ4gN3IurE0fpjxIkvHDiinQmk51BI7zQD8k1znU7r/spPqB+vZjc5ep6DC5wZUpFu5vJ8MoNKjCu8wnzyCFdA==
TyyOpen-Serial: 5157F09EFDC096DE15EBE81A47057A7232F1B8E1

{"data":[{"serial_no":"5157F09EFDC096DE15EBE81A47057A7232F1B8E1","effective_time":"2018-03-26T11:39:50+08:00","expire_time":"2023-03-25T11:39:50+08:00","encrypt_certificate":{"algorithm":"AEAD_AES_256_GCM","nonce":"4de73afd28b6","associated_data":"certificate","ciphertext":"..."}}]}
```
参数说明<br>
TyyOpen-Timestamp: 签名时间戳<br>
TyyOpen-Nonce: 签名随机字符<br>
TyyOpen-Signature: 签名结果, 需要三方平台做验签处理<br>
TyyOpen-Serial: 三方平台证书序号<br>

签名内容格式为 “签名时间戳 + \n + 签名随机字符 + \n + 报文主体 + \n”, 如下:
```
1627374437
150967fe0671410ba3a7525e5f5e2506
{"data":[{"serial_no":"5157F09EFDC096DE15EBE81A47057A7232F1B8E1","effective_time":"2018-03-26T11:39:50+08:00","expire_time":"2023-03-25T11:39:50+08:00","encrypt_certificate":{"algorithm":"AEAD_AES_256_GCM","nonce":"4de73afd28b6","associated_data":"certificate","ciphertext":"..."}}]}
```
签名处理示例:
```java
String pkcs8KeyPath = "PKCS8私钥路径";
String message = "签名时间戳" + "\n" + "签名随机字符" + "\n" + "报文内容" + "\n";
String sign = com.touscm.quicker.security.utils.SignatureUtils.sign(pkcs8KeyPath, message);
```

### 通用通知数据
```java
public class NotifyEntry {
    /**
     * 通知ID
     */
    @NotEmpty(message = "通知ID不能为空")
    private String id;
    /**
     * 通知创建时间
     */
    @JsonProperty("create_time")
    @JsonFormat(pattern = "YYYY-MM-dd'T'HH:mm:ss+08:00", timezone = "GMT+8")
    @NotNull(message = "通知创建时间不能为NULL")
    private Date createTime;
    /**
     * 通知类型
     */
    @JsonProperty("event_type")
    @NotEmpty(message = "通知类型不能为空")
    private String eventType;
    /**
     * 通知数据类型
     */
    @JsonProperty("resource_type")
    @NotEmpty(message = "通知数据类型不能为空")
    private String resourceType;
    /**
     * 通知数据
     */
    @NotNull(message = "通知数据不能为NULL")
    private ResourceEntry resource;
    /**
     * 回调摘要
     */
    @NotEmpty(message = "回调摘要不能为空")
    private String summary;
}
```

### 通知加密数据
```java
public class ResourceEntry {
    /**
     * 加密算法类型
     * 示例值: AEAD_AES_256_GCM
     */
    @NotEmpty(message = "加密算法类型不能为空")
    private String algorithm;
    /**
     * 数据密文
     */
    @NotEmpty(message = "数据密文不能为空")
    private String ciphertext;
    /**
     * 附加数据
     */
    @JsonProperty("associated_data")
    private String associatedData;
    /**
     * 加密使用的随机串
     */
    @NotEmpty(message = "随机串不能为空")
    private String nonce;
}
```

加密处理示例:
```java
String apiKey = "三方密钥";
String nonce = "随机16位字符-加密初始化向量";
String businessData = "业务数据";
String ciphertext = com.touscm.quicker.utils.EncryptUtils.aesGcmEncode(apiKey, nonce, businessData);
```

## 请求记录

### 注解使用
注解AccessRecord用于标记需要保存请求记录的接口, 可用于控制器类和控制器方法, 当标记控制器时需指定路由字段
```java
@RestController
@AccessRecord(route = "get", type = "请求类型", name = "请求名称")
public class Controller {
    @ControllerAction("testAction")
    @AccessRecord(type = "三方请求", name = "查询余额")
    public RestResult<T> testAction(@Valid @RequestBody ReqModel reqModel) {
        logger.info("程序进入查询站点余额方法,请求参数:{}", queryAccountParam);
        return accountService.getAccount(queryAccountParam);
    }
}
```

### 数据保存
实现IAccessRecordService接口的save方法
```java
@Service
public class AccessRecordService implements IAccessRecordService {
    /**
     * 请求记录保存
     *
     * @param accessType   请求记录类型
     * @param accessName   请求记录名称
     * @param method       请求方法
     * @param agent        终端类型
     * @param ip           请求IP地址
     * @param uri          请求地址
     * @param responseBody 返回内容
     * @param responseData 返回对象
     * @param userInfo     用户信息
     */
    public void save(String accessType, String accessName, String method, String agent, String ip, String uri, String responseBody, RestResult responseData, UserInfo userInfo) {
    }
}
```