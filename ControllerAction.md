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
}
```
`RestResult.status`的数值参照`RestResultStatus`枚举类
```java
public enum RestResultStatus {
    Success(200, "Success", "处理成功"),
    NoData(204, "NoData", "无数据"),
    ParamError(400, "ParamError", "参数错误"),
    NoAuth(401, "NoAuth", "未认证"),
    NoPermission(403, "NoPermission", "禁止访问"),
    NotFound(404, "NotFoundUrl", "路由出错"),
    NotAcceptable(406, "NotAcceptable", "禁止访问"),
    Failed(417, "Failed", "处理失败"),
    UserNameOrPasswordError(418, "UserNameOrPasswordError", "登录名或密码错误"),
    UserForbidden(419, "UserForbidden", "用户被禁用"),
    Error(500, "Error", "处理出错");
}
```

## 默认接口

`BaseController`默认已实现数个CRUD接口, 具体分类如下:
> * [ControllerAction.ADD](#ControllerAction.ADD)
> * [ControllerAction.EDIT](#ControllerAction.EDIT)
> * [ControllerAction.ACTIVE](#ControllerAction.ACTIVE)
> * [ControllerAction.DISABLE](#ControllerAction.DISABLE)
> * [ControllerAction.DELETE](#ControllerAction.DISABLE)
> * [ControllerAction.ANY](#ControllerAction.ANY)
> * [ControllerAction.GET](#ControllerAction.GET)
> * [ControllerAction.COUNT](#ControllerAction.COUNT)
> * [ControllerAction.LISTING](#ControllerAction.LISTING)
> * [ControllerAction.SELECTLIST](#ControllerAction.SELECTLIST)
> * [ControllerAction.PAGING](#ControllerAction.PAGING)
> * [ControllerAction.SUM](#ControllerAction.SUM)
> * [ControllerAction.REMOVE](#ControllerAction.REMOVE)

### ControllerAction.ADD

`ADD`接口用于添加数据, 路由为`/api/user/add`, 接收JSON数据, 格式为:
```json
{ "model": { "name": "UserName" }}
```
请求参数`model`对应控制器类定义的模型类, 此时为`UserModel`, curl请求示例如下
```shell
curl -X POST -H 'Content-Type:application/json' -H 'Authorization: Bearer xxx' \
  -d '{"model":{"name":"UserName"}}' \
  http://127.0.0.1:80/api/user/add
```
<label style='color:red'>注意</label>: `ADD`接口会进行参数校验, 关于数据格式校验请参照[数据格式校验说明](./Validation.md "数据格式校验说明")

### ControllerAction.EDIT

`EDIT`接口用于编辑数据, 路由为`/api/user/edit`, 接收JSON数据, 格式为:
```json
{ "model": { "id": "xxx", "name": "NewUserName" }}
```
请求参数`model`对应控制器类定义的模型类, 此时为`UserModel`, curl请求示例如下
```shell
curl -X POST -H 'Content-Type:application/json' -H 'Authorization: Bearer xxx' \
  -d '{"model":{"id":"xxx","name":"UserName"}}' \
  http://127.0.0.1:80/api/user/edit
```
<label style='color:red'>注意</label>: `edit`接口必须指定主键值, `EDIT`接口会进行参数校验, 关于数据格式校验请参照[数据格式校验说明](./Validation.md "数据格式校验说明")

### ControllerAction.ACTIVE

`ACTIVE`接口用于恢复逻辑删除的数据, 路由为`/api/user/active`, 处理逻辑为设置实体的`deleteStatus`字段值为`false`, 接收JSON数据, 格式为:
```json
{ "conditions": [{ "type": "id", "value": "xxx1" }, { "type": "id", "value": "xxx2" }]}
```
请求参数`conditions`为查询条件, 此时为主键列表, 可恢复逻辑删除多条数据, curl请求示例如下
```shell
curl -X POST -H 'Content-Type:application/json' -H 'Authorization: Bearer xxx' \
  -d '{"conditions":[{"type":"id","value":"xxx1"},{"type":"id","value":"xxx2"}]}' \
  http://127.0.0.1:80/api/user/active
```

### ControllerAction.DISABLE, ControllerAction.DELETE

`DISABLE`和`DELETE`接口用于逻辑删除数据, 路由为`/api/user/disable`和`/api/user/delete`, 处理逻辑为设置实体的`deleteStatus`字段值为`true`, 接收JSON数据, 格式为:
```json
{ "conditions": [{ "type": "id", "value": "xxx1" }, { "type": "id", "value": "xxx2" }]}
```
请求参数`conditions`为查询条件, 此时为主键列表, 可逻辑删除多条数据, curl请求示例如下
```shell
curl -X POST -H 'Content-Type:application/json' -H 'Authorization: Bearer xxx' \
  -d '{"conditions":[{"type":"id","value":"xxx1"},{"type":"id","value":"xxx2"}]}' \
  http://127.0.0.1:80/api/user/delete
```

### ControllerAction.ANY

`ANY`接口用于判断数据是否存在, 路由为`/api/user/any`, 接收JSON数据, 格式为:
```json
{ "conditions": [{ "type": "field1", "value": "value1" }, { "type": "field2", "value": "value2" }]}
```
查询条件参数`conditions`的使用可参照[查询条件说明](./Conditions.md "查询条件说明"), curl请求示例如下
```shell
curl -X POST -H 'Content-Type:application/json' -H 'Authorization: Bearer xxx' \
  -d '{"conditions":[{"type":"field1","value":"value1"},{"type":"field2","value":"value2"}]}' \
  http://127.0.0.1:80/api/user/any
```

### ControllerAction.GET

`GET`接口用于取得单条数据, 返回结果`data`字段为视图模型类, 路由为`/api/user/get`, 接收JSON数据, 格式为:
```json
{ "conditions": [{ "type": "username", "value": "admin" }, { "type": "deleteStatus", "value": "false" }]}
```
查询条件参数`conditions`的使用请参照 [查询条件说明](./Conditions.md "查询条件说明"), curl请求示例如下
```shell
curl -X POST -H 'Content-Type:application/json' -H 'Authorization: Bearer xxx' \
  -d '{ "conditions": [{ "type": "username", "value": "admin" }, { "type": "deleteStatus", "value": "false" }]}' \
  http://127.0.0.1:80/api/user/get
```
返回结果示例
```json
{
    "status": 200,
    "message": "处理成功",
    "messageDetails": null,
    "data": {
        "id": "15901325908815857588",
        "username": "admin",
        "password": "e10adc3949ba59abbe56e057f20f883e",
        "salt": null,
        "mobile": "18229051989",
        "createTime": "2020-05-22 15:29:51",
        "createBy": null,
        "updateTime": "2020-06-08 17:04:31",
        "updateBy": "15901325908815857588",
        "deleteStatus": false
    }
}
```
<label style='color:red'>注意</label>: `GET`接口在指定主键时将忽略其他请求条件

### ControllerAction.COUNT

`COUNT`接口用于统计数据条数, 返回结果`data`字段为长整型数值, 路由为`/api/user/count`, 接收JSON数据, 格式为:
```json
{ "conditions": [{ "type": "field1", "value": "value1" }, { "type": "field2", "value": "value2" }]}
```
查询条件参数`conditions`的使用请参照 [查询条件说明](./Conditions.md "查询条件说明"), curl请求示例如下
```shell
curl -X POST -H 'Content-Type:application/json' -H 'Authorization: Bearer xxx' \
  -d '{"conditions":[{"type":"field1","value":"value1"},{"type":"field2","value":"value2"}]}' \
  http://127.0.0.1:80/api/user/count
```
返回示例
```json
{"status":200,"message":"处理成功","messageDetails":null,"data":1097}
```
<label style='color:red'>注意</label>: `COUNT`接口会默认添加`delete_status=true`条件

### ControllerAction.LISTING

`LISTING`接口返回数据列表, 返回结果`data`字段为视图模型类列表, 路由为`/api/user/listing`, 接收JSON数据, 格式为:
```json
{ "conditions": [{ "type": "username", "value": "admin" }, { "type": "deleteStatus", "value": "false" }]}
```
查询条件参数`conditions`的使用请参照 [查询条件说明](./Conditions.md "查询条件说明"), curl请求示例如下
```shell
curl -X POST -H 'Content-Type:application/json' -H 'Authorization: Bearer xxx' \
  -d '{ "conditions": [{ "type": "username", "value": "admin" }, { "type": "deleteStatus", "value": "false" }]}' \
  http://127.0.0.1:80/api/user/count
```
返回示例
```json
{
    "status": 200,
    "message": "处理成功",
    "messageDetails": null,
    "data": [
        {
            "id": "15901325908815857588",
            "username": "admin",
            "password": "e10adc3949ba59abbe56e057f20f883e",
            "salt": null,
            "mobile": "18229051989",
            "createTime": "2020-05-22 15:29:51",
            "createBy": null,
            "updateTime": "2020-06-08 17:04:31",
            "updateBy": "15901325908815857588",
            "deleteStatus": false
        }
    ]
}
```
<label style='color:red'>注意</label>: `LISTING`接口会默认添加`delete_status=true`条件

### ControllerAction.SELECTLIST

`SELECTLIST`接口返回键值对列表, 返回结果`data`字段为`{ "key": "", "value": "" }`列表, 路由为`/api/user/selectlist`, 接收JSON数据, 格式为:
```json
{
	"keyValuePair": { "key": "id", "value": "username" },
	"conditions": [
		{ "type": "in_id", "value": "15901325908815857588" }, { "type": "in_id", "value": "16774604479413803319" }
	]
}
```
`keyValuePair`指定键值对对应实体的字段名, `conditions`指定查询条件列表, `conditions`的使用请参照 [查询条件说明](./Conditions.md "查询条件说明"), curl请求示例如下
```shell
curl -X POST -H 'Content-Type:application/json' -H 'Authorization: Bearer xxx' \
  -d '{ "conditions": [{ "type": "username", "value": "admin" }, { "type": "deleteStatus", "value": "false" }]}' \
  http://127.0.0.1:80/api/user/selectlist
```
返回示例
```json
{
    "status": 200,
    "message": null,
    "messageDetails": null,
    "data": [
        {
            "key": "15901325908815857588",
            "value": "admin"
        },
        {
            "key": "16774604479413803319",
            "value": "chenjian"
        }
    ]
}
```
<label style='color:red'>注意</label>: `LISTING`接口会默认添加`delete_status=true`条件

### ControllerAction.PAGING

`PAGING`接口返回键值对列表, 结果`data`字段为分页信息(分页索引,分页尺寸,总条数,页数,数据列表), 路由为`/api/user/paging`, 接收JSON数据, 格式为:
```json
{
	"page": 1, "size": 2, "conditions": [{ "type": "deleteStatus", "value": "false" }]
}
```
`page`指定分页页码, `size`指定分页尺寸, `conditions`指定查询条件列表, `conditions`的使用请参照 [查询条件说明](./Conditions.md "查询条件说明"), curl请求示例如下
```shell
curl -X POST -H 'Content-Type:application/json' -H 'Authorization: Bearer xxx' \
  -d '{ "page": 1, "size": 2, "conditions": [{ "type": "deleteStatus", "value": "false" }]}' \
  http://127.0.0.1:80/api/user/paging
```
返回示例
```json
{
    "status": 200,
    "message": "处理成功",
    "messageDetails": null,
    "data": {
        "pageIndex": 1,
        "pageSize": 2,
        "itemCount": 14,
        "pageCount": 7,
        "itemList": [
            {
                "id": "15901325908815857588",
                "username": "admin",
                "password": "e10adc3949ba59abbe56e057f20f883e",
                "salt": null,
                "mobile": "18229051989",
                "createTime": "2020-05-22 15:29:51",
                "createBy": null,
                "updateTime": "2020-06-08 17:04:31",
                "updateBy": "15901325908815857588",
                "deleteStatus": false
            },
            {
                "id": "16594207675436455039",
                "username": "liuzhe001",
                "password": "e10adc3949ba59abbe56e057f20f883e",
                "salt": "30327011b3494a61a88d05fcdc44f47c",
                "mobile": "13891351860",
                "createTime": "2022-08-02 14:12:48",
                "createBy": "15901325908815857588",
                "updateTime": "2023-08-16 12:59:36",
                "updateBy": "15901325908815857588",
                "deleteStatus": false
            }
        ]
    }
}
```
<label style='color:red'>注意</label>: `LISTING`接口会默认添加`delete_status=true`条件

### ControllerAction.SUM

`SUM`接口返回字段合计, 结果`data`字段为分页信息(分页索引,分页尺寸,总条数,页数,数据列表), 路由为`/api/user/sum`, 接收JSON数据, 格式为:
```json
{
	"conditions": [
		{ "type": "sum_balanceTotal", "value": "" },
		{ "type": "sum_rechargeAmoun", "value": "" },
		{ "type": "sum_consumptionAmount", "value": "" },
		{ "type": "sum_withdrawAmount", "value": "" },
		{ "type": "deleteStatus", "value": "false" }
	]
}
```
`SUM`接口需指定合计字段, 格式为`{ "type": "sum_${FieldName}", "value": "" }`, `${FieldName}`为实体字段名, 可指定多个字段<br>
`conditions`指定查询条件列表, `conditions`的使用请参照 [查询条件说明](./Conditions.md "查询条件说明"), curl请求示例如下
```shell
curl -X POST -H 'Content-Type:application/json' -H 'Authorization: Bearer xxx' \
  -d '{"conditions":[{"type":"sum_balanceTotal","value":""},{"type":"sum_rechargeAmoun","value":""},{"type":"sum_consumptionAmount","value":""},{"type":"sum_withdrawAmount","value":""},{"type":"deleteStatus","value":"false"}]}' \
  http://127.0.0.1:80/api/user/sum
```
返回示例
```json
{
    "status": 200,
    "message": "处理成功",
    "messageDetails": null,
    "data": {
        "id": null,
        "balanceTotal": 1305052.9800,
        "rechargeAmoun": 1339828.6600,
        "consumptionAmount": 17029.3900,
        "donationAmount": null,
        "withdrawAmount": 17758.9400,
        "transferInAmount": null,
        "lockStatus": null
    }
}
```
<label style='color:red'>注意</label>: `SUM`接口会根据查询的合计字段创建实体对象, 所以需创建对应的构造函数

### ControllerAction.REMOVE

`REMOVE`接口执行物理删除操作, 接收`DELETE`协议请求, 路由为`/${ControllerRoute}/${id}`, `${ControllerRoute}`为接口类路由, `${id}`为主键值, curl请求示例如下
```shell
curl -X DELETE -H 'Content-Type:application/json' -H 'Authorization: Bearer xxx' \
  -d '{}' \
  http://127.0.0.1:80/api/user/xxx
```
返回示例
```json
{"status":200,"message":"处理成功","messageDetails":null,"data":null}
```
<label style='color:red'>注意</label>: `REMOVE`接口为危险操作, 如非需要请添加`@Permission(denies = "ControllerAction.REMOVE")`注解至控制器类可禁用该接口

# >>>>>>

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