# 认证

## 处理范围

1. Spring MVC接口, 接口添加了`Permission`注解且未指定`isPublic`或者`publics`参数, 接口请求会进行认证检查
2. BaseController的默认接口和`ControllerAction`接口默认是需要认证检查的

## 处理机制

认证处理的核心是`com.touscm.quicker.security.handler.IAuthHandler`接口, 处理逻辑是在收到请求时, 拦截器会拿到请求头`Authorization`的值调用`IAuthHandler.validateToken()`方法识别用户, 保存用户信息到线程变量, 根据接口实际权限定义进行认证检查, `IAuthHandler.validateToken()`方法有3个重载

```java
    /**
     * 登录信息验证
     *
     * @param authContent 登录信息
     * @return 用户信息
     */
    Optional<UserInfo> validateToken(String authContent) {
    }
```
需要`Authorization`请求头完成用户认证
```java
    /**
     * 登录信息验证
     *
     * @param method      请求方法
     * @param uri         请求路由
     * @param authContent 登录信息
     * @param body        请求内容
     * @return 用户信息
     */
    Optional<UserInfo> validateToken(String method, String uri, String authContent, String body) {
    }
```
需要`Authorization`请求头, 请求方法, 请求路由, 请求体完成用户认证
```java
    /**
     * 登录信息验证
     *
     * @param method      请求方法
     * @param uri         请求路由
     * @param authContent 登录信息
     * @param body        请求内容
     * @param headers     请求头列表
     * @return 用户信息
     */
    Optional<UserInfo> validateToken(String method, String uri, String authContent, String body, Map<String, String> headers) {
    }
```
需要`Authorization`请求头, 请求方法, 请求路由, 请求体, 请求头列表完成用户认证<br>
根据实际需要重写相应方法

## 推荐实现

推荐采用Bearer Token方式进行认证, Bearer Token的标准请求方式如下
```ini
Authorization: Bearer [BEARER_TOKEN]
```
原始请求认证参数有group和token两个字段, group用来定义用户所属组织, token通常来自与登陆返回的令牌, 具体格式为
```json
{ "group": "USER-AUTH-GROUP", "token": "LOGIN-RES-TOKEN" }
```
使用Base64对该对原始认证参数进行编码
```shell
$ echo -n '{"group":"JWT", "token":"1234567890-1234567890-1234567890"}' | base64
$ eyJncm91cCI6IkpXVCIsICJ0b2tlbiI6IjEyMzQ1Njc4OTAtMTIzNDU2Nzg5MC0xMjM0NTY3ODkwIn0=
```
使用"Bearer "加上编码后的内容就是认证头内容
```text
Bearer eyJncm91cCI6IkpXVCIsICJ0b2tlbiI6IjEyMzQ1Njc4OTAtMTIzNDU2Nzg5MC0xMjM0NTY3ODkwIn0=
```
请求时把认证内容放到`Authorization`请求头里就可以访问接口, curl示例如下
```shell
curl -X POST -H 'Content-Type:application/json' \
  -H 'Authorization: Bearer eyJncm91cCI6IkpXVCIsICJ0b2tlbiI6IjEyMzQ1Njc4OTAtMTIzNDU2Nzg5MC0xMjM0NTY3ODkwIn0=' \
  -d '{"model":{"name":"name-value"}}' \
  http://127.0.0.1:80/api/sysRoleMenu/add
```

### 服务端认证处理

在登陆接口完成登陆后, 生成令牌并绑定令牌和用户关系, 然后返回令牌
```java
@Service
public class SessionService {
    @Resource
    private ICache<String> tokenCache;
    @Resource
    private IHashCache<String, UserInfo> sessionCache;

    @Value("${service.session-timeout-hour:24}")
    private int sessionTimeoutHour;

    /* ...... */

    public Optional<UserInfo> getSession(@NotNull GroupType groupType, @NotBlank String token) {
        return tokenCache.get(groupType.tokenKey(token)).flatMap(userId -> sessionCache.get(groupType.sessionKey(), userId));
    }

    public UserInfo saveSession(@NotNull User user) {
        String token = StringUtils.uuId(), cachedToken = null;
        UserInfo userInfo = sessionCache.get(GroupType.System.sessionKey(), user.getId()).orElse(null);
        List<GroupInfo> groups = Collections.singletonList(new GroupInfo<>().setId(GROUP_SYSTEM).setType(GROUP_SYSTEM));

        if (userInfo != null) {
            cachedToken = userInfo.getSessionToken();
            //noinspection unchecked
            userInfo.setName(user.getName()).setMobile(user.getPhone()).setSessionToken(token).setCurrentGroupId(GROUP_SYSTEM).setGroups(groups);
        } else {
            userInfo = new UserInfo<>();
            //noinspection unchecked
            userInfo.setId(user.getId()).setName(user.getName()).setMobile(user.getPhone()).setSessionToken(token).setCurrentGroupId(GROUP_SYSTEM).setGroups(groups);
        }

        // clear old token
        if (StringUtils.isNotEmpty(cachedToken)) {
            tokenCache.delete(GroupType.System.tokenKey(cachedToken));
        }

        tokenCache.set(GroupType.System.tokenKey(token), user.getId(), new Date(Instant.now().plus(sessionTimeoutHour, ChronoUnit.HOURS).toEpochMilli()));
        sessionCache.put(GroupType.System.sessionKey(), user.getId(), userInfo);

        return userInfo;
    }
}
```

### 服务端认证检查处理

实现`IAuthHandler`接口并重写`validateToken()`方法, 就可以接收认证请求处理
```java
public class TokenEntry {
    private String group;
    private String token;
}

@Service
public class AuthTokenHandler implements IAuthHandler {
    // 用于取得用户信息
    @Resource
    private SessionService sessionService;

    @Override
    public Optional<UserInfo> validateToken(String authContent) {
        if (!IAuthHandler.isBearerAuth(authContent)) return Optional.empty();

        String decodeStr = Base64Utils.decode(authContent.replace(AUTH_VALUE_PREFIX, EMPTY));
        if (StringUtils.isBlank(decodeStr)) return Optional.empty();

        TokenEntry entry = EntryUtils.parse(TokenEntry.class, decodeStr);
        if (entry == null || StringUtils.isEmpty(entry.getGroup()) || StringUtils.isBlank(entry.getToken())) return Optional.empty();

        // SessionService.getSession(), 根据实际需要实现用户信息取得逻辑
        Optional<UserInfo> session = GroupType.parse(entry.getGroup()).flatMap(groupType -> sessionService.getSession(groupType, entry.getToken()));

        // 如果能拿到用户信息, 控制器类和服务类就可以那个当前用户信息
        return session;
    }
}
```
认证检查完成后就可以在用户请求线程任何地方调用`SessionHandler.getSession()`取得登陆用户信息