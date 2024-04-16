# 数据访问

框架数据抽象层支持`JPA`和`mybatis`, 框架提供了数据抽象interface`IBaseDao`, `JPA`和`mybatis`实现都继自`IBaseDao`. 

## IBaseDao

`IBaseDao`定义了`count`(统计), `sum`(合计), `querySingle`(单条记录查询), `queryList`(查询列表), `queryPaging`(查询分页), `insert`(插入), `update`(更新), `delete`(删除)方法, 详情如下
```java
public interface IBaseDao<TEntity extends BaseEntry> {
    /**
     * query record count
     *
     * @param query query instance
     * @return record count
     */
    long count(@NotNull Query<TEntity> query);

    /**
     * query record count
     *
     * @param conditions req condition list
     * @return record count
     */
    long count(@NotEmpty List<ReqCondition> conditions);

    /**
     * query single sum field
     *
     * @param query  query statement
     * @param getter getter function
     * @param <V>    field Type
     * @return sum value
     */
    <V extends Number> V sumValue(@NotNull Query<TEntity> query, @NotNull Getter<TEntity, V> getter);

    /**
     * query multiple sum field
     *
     * @param query      query
     * @param resultType result type
     * @param <R>        result type
     * @return result
     */
    <R> Optional<R> sum(@NotNull Query<TEntity> query, @NotNull Class<R> resultType);

    /**
     * query multiple sum field
     *
     * @param conditions req condition list
     * @param resultType result type
     * @param <R>        result type
     * @return result
     */
    <R> Optional<R> sum(@NotEmpty List<ReqCondition> conditions, @NotNull Class<R> resultType);

    /**
     * query single result
     *
     * @param query query statement
     * @return optional entity
     */
    Optional<TEntity> querySingle(@NotNull Query<TEntity> query);

    /**
     * query single result
     *
     * @param conditions req condition list
     * @return optional entity
     */
    Optional<TEntity> querySingle(@NotEmpty List<ReqCondition> conditions);

    /**
     * query result list
     *
     * @param query query statement
     * @return result list
     */
    List<TEntity> queryList(@NotNull Query<TEntity> query);

    /**
     * query result list
     *
     * @param conditions request condition list
     * @return result list
     */
    List<TEntity> queryList(@NotEmpty List<ReqCondition> conditions);

    /**
     * query result paging
     *
     * @param query     query statement
     * @param pageIndex page index
     * @param pageSize  page size
     * @return result paging
     */
    PagingEntry<TEntity> queryPaging(@NotNull Query<TEntity> query, int pageIndex, int pageSize);

    /**
     * query result paging
     *
     * @param conditions request condition list
     * @param pageIndex  page index
     * @param pageSize   page size
     * @return result paging
     */
    PagingEntry<TEntity> queryPaging(@NotEmpty List<ReqCondition> conditions, int pageIndex, int pageSize);

    /**
     * insert record
     *
     * @param query       query statement
     * @param keyGenerate key value generate
     * @return insert row count
     */
    int insert(@NotNull Query<TEntity> query, @NotNull Consumer<String> keyGenerate);

    /**
     * insert record
     *
     * @param entity      entity
     * @param keyGenerate key value generate
     * @return insert row count
     */
    int insert(@NotNull TEntity entity, @NotNull Consumer<String> keyGenerate);

    /**
     * execute update query
     *
     * @param query query statement
     * @return effect row count
     */
    int update(@NotNull Query<TEntity> query);

    /**
     * execute delete query
     *
     * @param query query statement
     * @return delete row count
     */
    int delete(@NotNull Query<TEntity> query);
}
```
关于`List<ReqCondition> conditions`参数的使用请参照[查询条件](#Conditions.md), 关于`Query<TEntity> query`的使用请参照[Query说明](#Query)

## JPA

框架默认使用`JPA`进行数据访问, [实体类](#Entity.md)定义完成后就需要定义`DAO`接口, `JPA`接口需继承`com.touscm.quicker.data.jpa.IDao`,除了`IBaseDao`定义的方法, `JPA`接口提供了额外的数据访问方法, 定义如下
```java

/**
 * 数据操作接口
 */
@NoRepositoryBean
public interface IDao<TEntity extends BaseEntity> extends IBaseDao<TEntity>, JpaRepository<TEntity, String>, JpaSpecificationExecutor<TEntity> {
    /**
     * 数据存在判断
     *
     * @param entityType 实体类型
     * @param id         主键值
     * @return 判断结果
     */
    default boolean exist(@NotNull Class<TEntity> entityType, String id) {
        return 0 < this.count(Collections.singletonList(new ReqCondition(Constants.FIELD_ID, id)));
    }

    /**
     * 数据存在判断
     *
     * @param entityType 实体类型
     * @param conditions 查询条件
     * @return 判断结果
     */
    default boolean exist(@NotNull Class<TEntity> entityType, List<ReqCondition> conditions) {
        return 0 < this.count(conditions);
    }

    /* ...... */

    default boolean remove(List<String> ids) {
        List<TEntity> entities = findAllById(ids);
        if (isEmpty(entities)) return false;

        deleteAll(entities);
        return true;
    }

    default boolean remove(Class<TEntity> entityType, List<ReqCondition> conditions) {
        Specification<TEntity> spec = QueryUtils.getSpecification(entityType, conditions);
        if (spec == null) return false;

        List<TEntity> entities = findAll(spec);
        if (isEmpty(entities)) return false;

        deleteAll(entities);
        return true;
    }

    /* ...... */

    /**
     * 取得实体对象
     *
     * @param id ID
     * @return 实体对象
     */
    default Optional<TEntity> getEntity(@NotEmpty String id) {
        if (isEmpty(id)) return empty();
        return findById(id);
    }

    /**
     * 取得实体对象
     *
     * @param conditions 查询条件
     * @return 实体对象
     */
    default Optional<TEntity> getEntity(List<ReqCondition> conditions) {
        if (CollectionUtils.isNotEmpty(conditions)) {
            return querySingle(conditions);
        }
        return Optional.empty();
    }

    /* ...... */

    /**
     * 取得数据
     *
     * @param id       ID
     * @param mapper   映射处理
     * @param <TModel> 模型类型
     * @return 取得结果
     */
    default <TModel extends BaseEntry> TModel get(String id, Function<TEntity, TModel> mapper) {
        if (isEmpty(id) || mapper == null) return null;
        return findById(id).map(mapper).orElse(null);
    }

    /**
     * 取得数据
     *
     * @param spec     查询定义对象
     * @param mapper   映射处理
     * @param <TModel> 模型类型
     * @return 取得结果
     */
    default <TModel extends BaseEntry> Optional<TModel> get(Specification<TEntity> spec, Function<TEntity, TModel> mapper) {
        if (spec == null || mapper == null) return empty();
        return findOne(spec).map(mapper);
    }

    /**
     * 取得数据
     *
     * @param conditions 查询条件
     * @param mapper     映射处理
     * @param <TModel>   模型类型
     * @return 取得结果
     */
    default <TModel extends BaseEntry> Optional<TModel> get(List<ReqCondition> conditions, Function<TEntity, TModel> mapper) {
        if (isEmpty(conditions) || mapper == null) return empty();
        return this.getEntity(conditions).map(mapper);
    }

    /* ...... */

    /**
     * 取得实体列表
     *
     * @param entityType 实体类型
     * @param conditions 查询条件
     * @return 取得结果
     */
    default List<TEntity> getEntityList(Class<TEntity> entityType, List<ReqCondition> conditions) {
        Assert.notNull(entityType, "实体类型不能为NULL");
        return findAll(QueryUtils.getSpecification(entityType, conditions));
    }

    /* ...... */

    /**
     * 取得模型列表
     *
     * @param type       实体类型
     * @param conditions 查询条件
     * @param mapper     映射处理
     * @param <TModel>   实体类型
     * @return 模型列表
     */
    default <TModel extends BaseEntry> List<TModel> getList(Class<TEntity> type, List<ReqCondition> conditions, Function<TEntity, TModel> mapper) {
        return this.getList(type, conditions, mapper, false, false);
    }

    /**
     * 取得模型列表
     *
     * @param entityType           实体类型
     * @param conditions           查询条件
     * @param mapper               映射处理
     * @param isAsc                默认排序字段是否升序
     * @param ignoreEmptyCondition 是否忽略空查询条件
     * @param <TModel>             模型类型
     * @return 模型列表
     */
    default <TModel extends BaseEntry> List<TModel> getList(Class<TEntity> entityType, List<ReqCondition> conditions, Function<TEntity, TModel> mapper, boolean isAsc, boolean ignoreEmptyCondition) {
        Objects.requireNonNull(entityType, "未指定实体类型");
        Objects.requireNonNull(mapper, "未指定类型映射处理");

        Specification<TEntity> specification = QueryUtils.getSpecification(entityType, conditions);
        if (specification == null && !ignoreEmptyCondition) return Collections.emptyList();

        return findAll(specification, QueryUtils.getSort(entityType, conditions, isAsc)).stream().map(mapper).collect(toList());
    }

    /* ...... */

    default <TModel extends BaseEntry> List<TModel> getList(Class<TModel> modelType, Specification<TEntity> specification) {
        Objects.requireNonNull(modelType);
        return this.getList(specification, a -> a.mapping(modelType), null);
    }

    default <TModel extends BaseEntry> List<TModel> getList(Specification<TEntity> specification, Function<TEntity, TModel> mapper, List<OrderByInfo> orderByInfos) {
        Objects.requireNonNull(specification, "未指定查询定义对象");
        Objects.requireNonNull(mapper, "未指定类型映射处理");

        Sort sort = QueryUtils.getSort(orderByInfos);
        if (sort == null) return findAll(specification).stream().map(mapper).collect(toList());

        return findAll(specification, sort).stream().map(mapper).collect(toList());
    }

    /* ...... */

    /**
     * SUM查询
     *
     * @param modelType  模型类型
     * @param conditions 查询条件
     * @param <TModel>   模型类型
     * @return 查询结果
     */
    <TModel extends BaseEntry> TModel sum(Class<TModel> modelType, List<ReqCondition> conditions);

    /**
     * SUM查询
     *
     * @param modelType  模型类型
     * @param conditions 查询条件
     * @param sumQueries SUM查询信息
     * @param <TModel>   模型类型
     * @return 查询结果
     */
    <TModel extends BaseEntry> TModel sum(Class<TModel> modelType, List<ReqCondition> conditions, List<QueryFieldInfo> sumQueries);

    /* ...... */

    /**
     * 取得虚实体对象
     *
     * @param entryType  虚实体类型
     * @param conditions 查询条件
     * @param <TEntry>   实体类型
     * @return 虚实体对象
     */
    <TEntry extends BaseEntry> Optional<TEntry> getEntry(Class<TEntry> entryType, List<ReqCondition> conditions);

    /**
     * 取得虚实体列表
     *
     * @param entryType  虚实体类型
     * @param conditions 查询条件
     * @param size       分页尺寸
     * @param <TEntry>   实体类型
     * @return 虚实体列表
     */
    <TEntry extends BaseEntry> List<TEntry> getEntryList(Class<TEntry> entryType, List<ReqCondition> conditions, int size);

    /**
     * 取得虚实体分页对象
     *
     * @param entryType  虚实体类型
     * @param conditions 查询条件
     * @param page       页码
     * @param size       分页尺寸
     * @param <TEntry>   虚实体类型
     * @return 虚实体分页对象
     */
    <TEntry extends BaseEntry> PagingEntry<TEntry> getEntryPaging(Class<TEntry> entryType, List<ReqCondition> conditions, int page, int size);
}
```

`JPA`接口定义示例
```java
@Repository
public interface AccessLogDao extends com.touscm.quicker.data.jpa.IDao<AccessLog> {
}
```

`DAO`定义完成后需要项目启动类上添加实体扫描`@EntityScan(basePackages = {"com.acme.demo.entity"})`和`DAO`扫描`@EnableJpaRepositories(basePackages = {"com.acme.demo.dao"}, repositoryFactoryBeanClass = BaseDaoFactoryBean.class)`

## Mybatis

`Mybatis`接口需继承`com.touscm.quicker.data.mybatis.IDao`, 这样`DAO`就可以通过`IBaseDao`定义的方法实现数据处理, 无需定义xml文件, 复杂查询的xml定义这里不做赘述<br>
><label style='color:red'>注意</label>: `Mybatis`接口不能包含名为`count`, `sumValue`, `sum`, `querySingle`, `queryList`, `queryPaging`, `insert`, `update`, `delete`的方法

`Mybatis`接口定义示例
```java
@javax.persistence.Table(name = "my_table_name")
public interface MyDao extends com.touscm.quicker.data.mybatis.IDao<MyEntity> {
}
```

`Mybatis`接口扫描定义为`@com.touscm.quicker.data.mybatis.spring.annotation.MapperScan("com.acme.demo.mapper")`

## Query

`Query`对象为数据处理的抽象, 实体了实体的CURD操作, `Query`的泛型参数为实体类型.

#### 查询示例
```java
public class MyClass {
    @Resource UserDao dao;

    // 查询单条记录, 查询条件为 id='id' and name='name'
    public void testQuerySingle() {
        Optional<User> result;

        result = dao.querySingle(new Query<User>().where(User::getId, "id").and(User::getName, "name"));
        System.out.println(EntryUtils.toString(result));

        result = dao.querySingle(Arrays.asList(new ReqCondition("id", "id"), new ReqCondition("name", "name")));
        System.out.println(EntryUtils.toString(result));
    }

    // 指定检索字段单记录查询, 查询id, password字段
    public void testQuerySingle() {
        Optional<User> result;

        result = dao.querySingle(new Query<User>().select(User::getId).select(User::getPassword).where(User::getId, "id").and(User::getName, "name"));
        System.out.println(EntryUtils.toString(result));

        result = dao.querySingle(Arrays.asList(new ReqCondition("id", "id"), new ReqCondition("name", "name")));
        System.out.println(EntryUtils.toString(result));
    }

    // 查询记录列表, 查询条件为 id='id' and name='name'
    public void testQueryList() {
        List<User> result;

        result = dao.queryList(new Query<User>().where(User::getId, "id").and(User::getName, "name"));
        System.out.println(EntryUtils.toString(result));

        result = dao.queryList(Arrays.asList(new ReqCondition("id", "id"), new ReqCondition("name", "name")));
        System.out.println(EntryUtils.toString(result));
    }

    // 查询分页, 查询条件为 id='id' and name='name'
    public void testQueryPaging() {
        PagingEntry<User> result;
        int page = 1, pageSize = 10;

        result = dao.queryPaging(new Query<User>().where(User::getId, "id").and(User::getName, "name"), page, pageSize);
        System.out.println(EntryUtils.toString(result));

        result = dao.queryPaging(Arrays.asList(new ReqCondition("id", "id"), new ReqCondition("name", "name")), page, pageSize);
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

        count = dao.count(new Query<MyEntity>().where(MyEntity::getId, "id").and(MyEntity::getName, "name"));
        System.out.println(count);

        count = dao.count(Arrays.asList(new ReqCondition("id", "id"), new ReqCondition("name", "name")));
        System.out.println(count);
    }

    public void testSumValue() {
        long sum = dao.sumValue(new Query<MyEntity>().where(MyEntity::getId, "id").and(MyEntity::getName, "name"), User::getValue);
        System.out.println(sum);
    }

    public void testSum() {
        Optional<MyEntity> sum = dao.sum(new Query<MyEntity>().sumSelect(MyEntity::getValue).where(MyEntity::getName, "name"), MyEntity.class);
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

        insertCount = dao.insert(new Query<MyEntity>().set(MyEntity::setId, "id").set(MyEntity::setName, "name"));
        System.out.println(insertCount);

        insertCount = dao.insert(new MyEntity().setId("id").setName("name"));
        System.out.println(insertCount);
    }
}
```

#### 更新示例
```java
    public void testUpdate(UserDao dao) {
        int updateCount;

        updateCount = dao.update(new Query<>(User::getUsername, "name1").set(User::setUpdateName, new Date().toString()));
        System.out.println(updateCount);

        dao.querySingle(new Query<>(User::getUsername, "name2")).ifPresent(entity -> {
            System.out.println(dao.update(new Query<>(User::getId, entity.getId()).set(User::setUsername, new Date().toString())));
        });
    }
```

#### 添加示例
```java
    public void testInsert(UserDao dao) {
        int insertCount;
        AtomicReference<String> keyRef = new AtomicReference<>();

        dao.delete(new Query<>(User::getUsername, "name1"));
        dao.delete(new Query<>(User::getUsername, "name2"));

        User entity = new User();
        entity.setUsername("name1").setPassword("password").setCreateDate(new Date());
        insertCount = dao.insert(entity, keyRef::set);
        System.out.println(insertCount + " " + keyRef.get());

        insertCount = dao.insert(new Query<User>().set(User::setUsername, "name2").set(User::setPassword, "password").set(User::setCreateDate, new Date()), keyRef::set);
        System.out.println(insertCount + " " + keyRef.get());
    }
```