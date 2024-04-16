# 数据实体

实体类是数据的抽象, 一般情况下实体类与数据库表一致. 框架支持`JPA`和`mybatis`, 框架默认使用`JPA`, 框架要求`JPA`实体类继承于`BaseEntity`或`GenericEntity`. `BaseEntity`是空类未包含字段, `GenericEntity`包含了一些预定义的字段, 并且主键为`UUID`, 可以根据实际需要定义实体基类. `mybatis`实体类未做限制.

## BaseEntity

如果表字段与`GenericEntity`不一致或者主键非`UUID`那么实体类需继承`BaseEntity`. `UUID`主键在大量数据时查询性能会出现问题, 并且数据分布不规律, 如果想避免这种情况发生可使用雪花算法住主键, 示例如下
```java
@Entity(name = "t_user_long")
public class User extends BaseEntity {
    @Id
    @GeneratedValue(generator = "snowflake")
    @GenericGenerator(name = "snowflake", strategy = "com.touscm.quicker.data.jpa.id.SnowflakeGenerator")
    protected long id;

    @Column(length = 50)
    private String username;
    private String password;

    private String createBy;
    private String updateBy;
    private String createName;
    private String updateName;
    private Date createDate;
    private Date updateDate;

    public long getId() {
        return id;
    }
    public void setId(long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }
    public User setUsername(String username) {
        this.username = username;
        return this;
    }

    public String getPassword() {
        return password;
    }
    public User setPassword(String password) {
        this.password = password;
        return this;
    }

    public String getCreateBy() {
        return createBy;
    }
    public void setCreateBy(String createBy) {
        this.createBy = createBy;
    }

    public String getUpdateBy() {
        return updateBy;
    }
    public void setUpdateBy(String updateBy) {
        this.updateBy = updateBy;
    }

    public String getCreateName() {
        return createName;
    }
    public void setCreateName(String createName) {
        this.createName = createName;
    }

    public String getUpdateName() {
        return updateName;
    }
    public void setUpdateName(String updateName) {
        this.updateName = updateName;
    }

    public Date getCreateDate() {
        return createDate;
    }
    public void setCreateDate(Date createTime) {
        this.createDate = createTime;
    }

    public Date getUpdateDate() {
        return updateDate;
    }
    public void setUpdateDate(Date updateTime) {
        this.updateDate = updateTime;
    }
}
```

## GenericEntity

`GenericEntity`定义如下
```java
public class GenericEntity extends BaseEntity {
    @Id
    @GeneratedValue(generator = "uuid")
    @GenericGenerator(name = "uuid", strategy = "com.touscm.quicker.data.jpa.id.UUIDGenerator")
    protected String id;
    protected String createBy;
    protected String updateBy;
    protected String createName;
    protected String updateName;
    protected Date createTime;
    protected Date updateTime;
    protected Boolean deleteStatus;

    public GenericEntity() {
        this.deleteStatus = false;
    }

    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }

    public String getCreateBy() {
        return this.createBy;
    }
    public void setCreateBy(String createBy) {
        this.createBy = createBy;
    }

    public String getUpdateBy() {
        return this.updateBy;
    }
    public void setUpdateBy(String updateBy) {
        this.updateBy = updateBy;
    }

    public String getCreateName() {
        return createName;
    }
    public void setCreateName(String createName) {
        this.createName = createName;
    }

    public String getUpdateName() {
        return updateName;
    }
    public void setUpdateName(String updateName) {
        this.updateName = updateName;
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

    public Boolean getDeleteStatus() {
        return deleteStatus;
    }
    public void setDeleteStatus(Boolean deleteStatus) {
        this.deleteStatus = deleteStatus;
    }
}
```

`GenericEntity`的主键为`UUID`类型, 所以数据库字段长度需为32字符类型, 字段`createBy`, `createName`, `createTime`在新增处理时维护, `updateBy`, `updateName`, `updateTime`会在更新处理时维护, `deleteStatus`字段为逻辑删除标识字段, 默认接口`ControllerAction.ACTIVE`处理会把该字段值设置为`false`, `ControllerAction.DISABLE`和`ControllerAction.DELETE`处理会把`deleteStatus`字段设值为`true`
