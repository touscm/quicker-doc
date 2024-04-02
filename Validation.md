# 数据验证

接口在接收到请求时往往需要验证请求参数才能进行业务处理, 框架支持注解和自定义数据验证

## 注解数据验证

注解数据验证包含标准JSR303验证和框架验证注解

### JSR303

JSR303属于Java EE规范, 其包含注解如下
* `@Null` 验证对象是否为NULL
* `@NotNull` 验证对象是否不为NULL
* `@NotBlank` 验证String对象不为NULL且不全为空格
* `@NotEmpty` 验证数组或列表不为空
* `@Size` 验证对象(Array,Collection,Map,String)长度
* `@Length` 验证String对象长度
* `@Pattern` 验证String正则表达式
* `@Email` 验证String对象是否符合电子邮件格式
* `@Min` 验证数字或String最小值
* `@Max` 验证数字或String最大值
* `@AssertTrue` 验证bool对象是否为True
* `@AssertFalse` 验证bool对象是否为False

### Validation注解

由于框架使用模型类用于多个接口, 例如默认接口`ADD`和`EDIT`都使用模型类作为入参, 两个处理的验证规则不同, 所以无法使用JSR303验证数据<br>
`@Validation`注解支持通过指定`policy`字段值进行不同的数据验证, 定义如下
```json
public @interface Validation {
    /**
     * 策略
     */
    String policy() default "";
    /**
     * 是否必须
     */
    boolean require() default true;
    /**
     * 消息名
     */
    String name() default "";
    /**
     * 验证错误信息
     */
    String message() default "";
    /**
     * 规则
     */
    ValidationRule rule() default ValidationRule.None;
    /**
     * 自定义校验规则
     */
    String regex() default "";
}
```
`policy`字段指定验证策略, `regex`字段指定正则表达式, `rule`字段自定预定义的一些验证类型, `rule`常用的值有
* `ValidationRule.Mobile` 手机号
* `ValidationRule.Captcha` 验证码
* `ValidationRule.IC` 组织机构代码证
* `ValidationRule.TIR` 道路运输许可证
* `ValidationRule.IdCard` 身份证
* `ValidationRule.CarNumber` 车牌号
* `ValidationRule.DriverLicenseNumber` 驾驶证号码

使用注解示例
```java
public class UserModel extends BaseModel {
    @Display("主键")
    @Validation(policy = "edit")
    @Validation(policy = "changePwd")
    private String id;

    @Display("用户名")
    @Validation(policy = "add", regex = "^[a-zA-Z][a-zA-Z0-9]{2,20}$")
    private String username;

    @Display("密码")
    @Validation(policy = "add")
    @Validation(policy = "changePwd", message = "当前密码不能为空")
    private String password;

    @Validation(policy = "changePwd", message = "新密码不能为空")
    private String newPassword;

    @Display("手机号")
    @Validation(policy = "add", rule = Validation.ValidationRule.Mobile, message = "手机号格式错误")
    private String mobile;

    @Display("姓名")
    @Validation(policy = "add", require = false, regex = "^[\\u0391-\\uFFE5]{2,5}$")
    private String name;
}
```
该示例表示当进行
* `add`验证时, `username`的非空正则验证, `password`的非空验证, `mobile`可为空, 有值时进行手机号格式验证, `name`可为空, 有值时进行2-5位中文验证
* `edit`验证时, `id`不能为空
* `changePwd`验证时, `id`不能为空, `password`不能为空, `newPassword`不能为空

## 数据验证

使用示例
```java
    @ControllerAction(ControllerAction.ADD)
    @Permission(isPublic = true)
    public RestResult<Object> doAdd(ReqModel<UserModel> reqModel) {
        UserModel model;
        ValidateResult validateResult = null;
        RestResultStatus status = RestResultStatus.ParamError;

        if (reqModel != null && (model = reqModel.getModel()) != null && (validateResult = model.validate("add")).isValidated()) {
            status = service.add(model, entity -> {
                entity.setSalt(StringUtils.uuId());
                entity.setPassword(EncryptUtils.sha256(entity.getSalt() + model.getPassword()));
            }) ? RestResultStatus.Success : RestResultStatus.Failed;
        }

        return new RestResult<>(status.getValue(), status.getMessage(), EntryUtils.valueOf(validateResult, ValidateResult::getMessages), null);
    }
```
基于`BaseModel`的类型调用`validate()`方法就可以进行数据验证, 也可以直接调用`EntryUtils.validate(modelObj, "policy")`方法

## 自定义验证规则

需要数值验证以外或者自定义验证时进行数据验证示例
```java
    @ControllerAction(ControllerAction.ADD)
    @Permission(isPublic = true)
    public RestResult<Object> doAdd(ReqModel<UserModel> reqModel) {
        UserModel model;
        ValidateResult validateResult = null;
        RestResultStatus status = RestResultStatus.ParamError;

        if (reqModel != null && (model = reqModel.getModel()) != null && (validateResult = model.validate("add", () -> {
            if (service.exist(Collections.singletonList(new ReqCondition("username", model.getUsername())))) {
                return new ValidateResult(false, new KeyValuePair<>("username", "用户已存在, 请修改用户名后重试"));
            }
            Optional<UserViewModel> userViewModel = service.get(List.of(new ReqCondition("mobile", model.getMobile())));
            if (userViewModel.isPresent()) {
                return new ValidateResult(false, new KeyValuePair<>("mobile", "用户手机号已存在, 请修改用户手机号后重试"););
            }
            return new ValidateResult(true);
        })).isValidated()) {
            status = service.add(model, entity -> {
                entity.setSalt(StringUtils.uuId());
                entity.setPassword(EncryptUtils.sha256(entity.getSalt() + model.getPassword()));
            }) ? RestResultStatus.Success : RestResultStatus.Failed;
        }

        return new RestResult<>(status.getValue(), status.getMessage(), EntryUtils.valueOf(validateResult, ValidateResult::getMessages), null);
    }
```
调用`public ValidateResult validate(String policy, Supplier<ValidateResult> checker)`重载方法就可以实现自定义验证规则, `checker`接收`ValidateResult`类型的返回结果