# 接口说明

框架控制器类需继承`com.touscm.quicker.web.BaseController`类, 并添加`@Controller`注解<br>
框架控制器类包含泛型参数, 其泛型参数依次为: 实体类, 模型类, 视图模型类, 服务类, 示例如下:
```java
@RestController
@RequestMapping("/api/user")
public class UserController extends BaseController<User, UserModel, UserViewModel, UserService> {
}
```
框架接口包括
> * [默认接口](#默认接口)
> * [自定义接口](#自定义接口)

默认接口接收统一的请求类型, 请求类型如下:
```java
public class ReqModel<TModel extends BaseModel> {
    /**
     * 页码
     */
    private int page;
    /**
     * 分页尺寸
     */
    private int size;
    /**
     * 请求模型
     */
    private TModel model;
    /**
     * 查询条件列表
     */
    private List<ReqCondition> conditions;
    /**
     * SelectList键值对
     */
    private KeyValuePair<String, String> keyValuePair;
}
```
默认接口返回类型也是统一的, 返回类型如下
```java
public class RestResult<TModel> {
    /**
     * 状态
     */
    private int status;
    /**
     * 信息
     */
    private String message;
    /**
     * 信息详情
     */
    private List<KeyValuePair<String, String>> messageDetails;
    /**
     * 数据模型
     */
    private TModel data;

    /* ...... */

    public RestResult() {
    }

    public RestResult(int status, TModel data) {
        this(status, null, null, data);
    }

    public RestResult(int status, String message, Map<String, String> messageMap) {
        this(status, message, null, null);
        List<KeyValuePair<String, String>> messageDetails = null;
        if (CollectionUtils.isNotEmpty(messageMap)) {
            messageDetails = messageMap.entrySet().stream().map(a -> new KeyValuePair<String, String>(a.getKey(), a.getValue())).collect(toList());
        }
        this.messageDetails = messageDetails;
    }

    public RestResult(int status, String message, List<KeyValuePair<String, String>> messageDetails) {
        this(status, message, messageDetails, null);
    }

    public RestResult(int status, String message, List<KeyValuePair<String, String>> messageDetails, TModel data) {
        if (StringUtils.isEmpty(message) && CollectionUtils.isNotEmpty(messageDetails)) {
            StringBuilder sb = new StringBuilder();
            messageDetails.forEach(a -> sb.append(sb.length() == 0 ? EMPTY : ";").append(String.format("%s=%s", a.getKey(), a.getValue())));
            message = String.format("数据验证失败, %s", sb);
        }

        this.status = status;
        this.message = message;
        this.messageDetails = messageDetails;
        this.data = data;
    }
}
```

## 默认接口

`BaseController`默认已实现数个CRUD接口, 具体分类如下:
> * [ControllerAction.ADD](#ControllerAction.ADD)
> * [ControllerAction.EDIT](#ControllerAction.EDIT)
> * [ControllerAction.ACTIVE](#ControllerAction.ACTIVE)
> * [ControllerAction.DISABLE](#ControllerAction.DISABLE)
> * [ControllerAction.DELETE](#ControllerAction.DELETE)
> * [ControllerAction.ANY](#ControllerAction.ANY)
> * [ControllerAction.GET](#ControllerAction.GET)
> * [ControllerAction.COUNT](#ControllerAction.COUNT)
> * [ControllerAction.LISTING](#ControllerAction.LISTING)
> * [ControllerAction.SELECTLIST](#ControllerAction.SELECTLIST)
> * [ControllerAction.PAGING](#ControllerAction.PAGING)
> * [ControllerAction.SUM](#ControllerAction.SUM)
> * [ControllerAction.REMOVE](#ControllerAction.REMOVE)

### ControllerAction.ADD

`ADD`接口用于添加记录, 路由为`/api/user/add`, 其接收JSON数据格式为:
```json
{
	"model": {
		"name": "UserName"
	}
}
```
请求参数`model`对应控制器类定义的模型类, 此时为`UserModel`

### ControllerAction.EDIT

`EDIT`接口用户编辑记录, 路由为`/api/user/edit`, 其接收JSON数据格式为:
```json
{
	"model": {
		"id": "EntityId", "name": "NewUserName"
	}
}
```
请求参数`model`对应 `Controller` 定义的模型类.<br/>

`deleteStatus` 方法, 路由为 "Controller/Delete", 其接收 JSON 数据格式为:
```json
{
	"conditions": [
		{ "type": "GoodsTemplateId", "value": "48780417-957c-11e8-af7d-f0def19cb946" }
	]
}
```
其中 `conditions` -> `type` 的值为 实体类类名+实体类主键字段名, `conditions` -> `value` 的值为 主键的值.<br>
删除操作可包含多个参数<br>
<label style='color:red'>注意</label>:实体类需包含 `isDelete` 字段删除操作才能正常工作.<br>
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
	"status": 200,
	"message": "处理成功",
	"data": null
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