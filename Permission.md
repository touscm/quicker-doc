# 权限

## 接口权限

定义接口权限需使用`com.touscm.quicker.base.Permission`注解, `Permission`注解可以添加到控制器类和接口方法<br>
<label style='color:red'>注意</label>: 对于Spring原生接口方法, 如果未定义`Permission`注解, 则认为该接口为公开接口<br>
<label style='color:red'>注意</label>: 对于ControllerAction接口方法, 如果未定义`Permission`注解, 则认为该接口为需认证接口, 不做权限检查<br>
当`Permission`注解添加到控制器类时需指定`route`字段, 该字段表示要作用的路由, `Permission`注解添加到接口方法时不指定`route`字段<br>

### 接口权限分类

1. **公开接口**
2. **禁用接口**
3. **内部接口**
4. **权限接口**

### 公开接口

公开接口需要对外公开, 并不做登录检查, 公开单路由时指定`Permission`的`isPublic`字段为`true`, 公开多路由时指定`Permission`的`publics`为公开的路由列表<br>
控制器类单独公开`/api/carrie/get`和`/api/carrie/listing`接口
```java
@RequestMapping("/api/carrie")
@Permission(route = GET, isPublic = true)
@Permission(route = LISTING, isPublic = true)
public class CarrieController extends BaseController<Carrie, CarrieModel, CarrieViewModel, CarrieService> {
}
```
控制器类批量公开`/api/carrie/get`和`/api/carrie/listing`接口
```java
@RequestMapping("/api/carrie")
@Permission(publics = {GET, LISTING, PAGING})
public class CarrieController extends BaseController<Carrie, CarrieModel, CarrieViewModel, CarrieService> {
}
```
接口方法上公开路由
```java
@RequestMapping("/api/carrie")
public class CarrieController extends BaseController<Carrie, CarrieModel, CarrieViewModel, CarrieService> {
    @ControllerAction(route = GET)
    @Permission(isPublic = true)
    public RestResult<?> addCarrie(ReqModel<CarrieModel> reqModel) {
    }
}
```

### 禁用接口

因为`BaseController`会提供数个默认接口, 当需要屏蔽默认接口时, 可以指定`Permission`的`isDenied`字段为`true`, 或者添加路由到`Permission`的`denies`字段, 示例如下:

```java
@RestController
@RequestMapping("/api/carrie")
@Permission(denies = {ADD, EDIT, DELETE, ACTIVE, DISABLE, REMOVE})
public class CarrieController extends BaseController<Carrie, CarrieModel, CarrieViewModel, CarrieService> {
}
```
该示例禁用`/api/carrie/add`等多个接口
```java
@RestController
@RequestMapping("/api/log")
@Permission(route = ADD, isDenied = true)
@Permission(route = EDIT, isDenied = true)
public class SysLogController extends BaseController<SysLog, SysLogModel, SysLogViewModel, SysLogService> {
}
```
该示例禁用`/api/log/add`和`/api/log/edit`接口

### 内部接口

内部接口仅限于服务之间的通讯, 需指定`isInternal`字段为`true`, 示例如下:

```java
@RestController
@RequestMapping("/api/log")
@Permission(action = ADD, isInternal = true)
@Permission(action = EDIT, isInternal = true)
public class SysLogController extends BaseController<SysLog, SysLogModel, SysLogViewModel, SysLogService> {
}
```
该示例将`/api/log/add`和`/api/log/edit`接口置为内部接口, 内部接口只能通过基础服务类`BaseService`提供的方法`sendServiceRest()`访问

### 权限接口

权限配置可分为静态和动态配置两种, 权限接口根据组织, 角色信息配置时为静态配置, `Permission`注解静态权限配置字段详述:
>`groups`字段定义登陆用户当前组织ID<br>
>`groupTypes`字段定义登陆用户当前组织类型<br>
>`roles`字段定义登陆用户当前组织包含的角色ID<br>
>`roleTypes`字段定义登陆用户当前组织包含的角色类型<br>

```java
@RestController
@RequestMapping("/api/sysUser")
@Permission(route = GET, groupTypes = {"System"})
@Permission(route = PAGING, groupTypes = {"System"})
public class SysUserController extends BaseController<SysUser, SysUserModel, SysUserViewModel, SysUserService> {
}
```
该示例定义`/api/sysUser/get`和`/api/sysUser/paging`接口的权限为组织类型为`System`的登陆用户访问
```java
@RestController
@RequestMapping("/api/sysUser")
@Permission(route = GET, groupTypes = {"System"}, roleTypes = {"Manager"})
@Permission(route = PAGING, groupTypes = {"System"}, roleTypes = {"Manager"})
public class SysUserController extends BaseController<SysUser, SysUserModel, SysUserViewModel, SysUserService> {
}
```
该示例定义`/api/sysUser/get`和`/api/sysUser/paging`接口的权限为组织类型为`System`并且角色类型为`Manager`的登陆用户访问

权限接口根据操作配置权限时为动态配置, `Permission`注解静态权限配置字段详述:
>`actions`字段定义操作权限, 查找逻辑为: 用户绑定角色 -> 角色绑定权限 -> 权限操作列表

```java
@RestController
@RequestMapping("/api/sysUser")
@Permission(route = ADD, groupTypes = {"System"}, actions = {"sys:user:add"})
@Permission(route = GET, groupTypes = {"System"}, actions = {"sys:user:view"})
public class SysUserController extends BaseController<SysUser, SysUserModel, SysUserViewModel, SysUserService> {
}
```
该示例定义`/api/sysUser/add`接口的权限为组织类型为`System`并且配置操作权限为`sys:user:add`的登陆用户访问

## 数据权限

数据权限用于限定数据所属, 数据权限分为查询权限和插入权限, 数据权限配置可分为全局配置和实体单独配置

### 全局配置数据权限配置

全局数据权限配置需要继承`com.touscm.quicker.service.permission.IPermissionService`接口<br>
实现查询权限需重写该接口的`getPermissionCondition()`方法<br>
实现插入权限需重写该接口的`setEntityPermissionFields()`方法<br>
编辑时不允许修改的字段可以在字段添加`com.touscm.quicker.data.annotations.EditIgnore`注解

```java
@Primary
@Service
public class PermissionService extends BasePermissionService {
    @Override
    public List<ReqCondition> getPermissionCondition(Optional<UserInfo> session, Class<?> entityType, List<ReqCondition> conditions) {
        return session.flatMap(userInfo -> {
            return ((Optional<GroupInfo>) userInfo.getCurrentGroup()).flatMap(groupInfo -> {
                var groupType = GroupType.parse(groupInfo.type());
                if (groupType != null) {
                    switch (groupType) {
                        case SP:
                            return Optional.of(getSpConditions(entityType, groupInfo, userInfo));
                        case Company:
                            return Optional.of(getCompanyConditions(entityType, groupInfo, userInfo));
                    }
                }
                return Optional.empty();
            });
        }).orElse(Collections.emptyList());
    }

    private List<ReqCondition> getSpConditions(Class<?> entityType, GroupInfo groupInfo, UserInfo userInfo) {
        return Collections.singletonList(new ReqCondition("spId", groupInfo.getId()));
    }

    private List<ReqCondition> getCompanyConditions(Class<?> entityType, GroupInfo groupInfo, UserInfo userInfo) {
        return Collections.singletonList(new ReqCondition("companyId", groupInfo.getId()));
    }
}
```
该示例实现在SP组织类型的用户在查询时给请求条件里附加`spId=user.groupId`条件, Company组织类型的用户在查询数据时附加`companyId=user.groupId`条件

```java
@Primary
@Service
public class PermissionService extends BasePermissionService {
    @Override
    public <TEntity extends BaseEntry> void setEntityPermissionFields(Optional<UserInfo> session, TEntity entity) {
        session.ifPresent(userInfo -> {
            ((Optional<GroupInfo>) userInfo.getCurrentGroup()).ifPresent(groupInfo -> {
                GroupType.parseOptional(groupInfo.type()).ifPresent(groupType -> {
                    switch (groupType) {
                        case Company:
                            setCompanyPermissionField(entity, groupInfo, userInfo);
                            break;
                    }
                });
            });
        });
    }

    private <TEntity extends BaseEntry> void setCompanyPermissionField(TEntity entity, GroupInfo groupInfo, UserInfo userInfo) {
        if (entity instanceof CompanyUser) {
            ((CompanyUser) entity).setCompanyId(groupInfo.getId());
            ((CompanyUser) entity).setCompanyName(groupInfo.getName());
        }
        if (entity instanceof CompanyRole) {
            ((CompanyRole) entity).setCompanyId(groupInfo.getId());
            ((CompanyRole) entity).setCompanyName(groupInfo.getName());
        }
        if (entity instanceof CompanyMenu) {
            ((CompanyMenu)entity).setCompanyId(groupInfo.getId());
            ((CompanyMenu)entity).setCompanyName(groupInfo.getName());
        }
    }
}
```
该示例实现在Company组织类型的用户在插入/编辑数据, 并且实体类型为`CompanyUser`, `CompanyRole`或者`CompanyMenu`时设置实体`companyId`字段值为`user.groupId`, `companyName`字段值为`user.groupName`

### 实体范围数据权限配置

定义实体的数据权限可在实体字段定义`com.touscm.quicker.data.annotations.Affiliation`注解<br>
`Affiliation`注解会覆盖全局数据权限配置, 并且对于查询和插入都会生效<br>

`Affiliation.ProviderCategory`分类:
>`SessionUserId`登陆用户的用户ID<br>
>`SessionGroupId`登陆用户的当前组织ID<br>
>`PermissionService`字段值由`IPermissionService.getPermissionCondition()`提供

```java
@Entity(name = "t_carrie")
@DynamicInsert
@DynamicUpdate
public class Carrie extends GenericEntity {
    @Affiliation(Affiliation.ProviderCategory.SessionGroupId)
    private String spId;
    @Affiliation(Affiliation.ProviderCategory.PermissionService)
    private String spName;
    @Affiliation(Affiliation.ProviderCategory.SessionUserId)
    private String companyId;
}

@Primary
@Service
public class CustomPermissionService extends BasePermissionService {
    @Override
    public Optional<ReqCondition> getPermissionCondition(@NotNull Optional<UserInfo> session, @NotNull Class<?> entityType, @NotNull Field field) {
        if (entityType == Carrie.class && "spName".equals(field.getName())) {
            return Optional.of(new ReqCondition("spName", "1"));
        }
        return Optional.empty();
    }
}
```
该示例表示在查询`Carrie`实体时会附加`spId=${user.groupId}`, `companyId=${user.id}`和`spName=${CustomPermissionService.getPermissionCondition()}`

## 动态权限

