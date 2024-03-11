# 查询条件说明

框架支持通过前端指定查询条件检索数据, 请求参数`conditions`为`com.touscm.quicker.domain.web.ReqCondition`列表,  定义如下
```java
public class ReqCondition {
    /**
     * 查询类型
     */
    private String query;
    /**
     * 查询单元类型
     */
    private String type;
    /**
     * 查询单元数值
     */
    private String value;
    /**
     * 子查询列表
     */
    private List<ReqCondition> conditions;
}
```
`query`用于指定子查询列表的逻辑关系, 默认为`AND`关系, `type`用于定义查询单元类型, `value`指定查询单元值, `conditions`定义子查询列表, 通过指定`conditions`的层次结构可实现简单查询和复杂查询

## 查询单元类型

`ReqCondition.type`作用有两个
> * [定义查询单元逻辑关系](#QueryRelation "查询单元逻辑关系说明")
> * [定义查询字段或者查询类型](#查询字段或者查询类型说明 "查询字段或者查询类型说明")

### QueryRelation

查询单元逻辑关系由枚举`com.touscm.quicker.data.QueryRelation`定义
```java
public enum QueryRelation {
    /**
     * 相等
     */
    Equals("Equals"),
    /**
     * 不相等
     */
    NotEquals("NotEquals"),
    /**
     * 大于
     */
    Great("Great"),
    /**
     * 大于等于
     */
    GreatEquals("GreatEquals"),
    /**
     * 小于
     */
    Less("Less"),
    /**
     * 小于等于
     */
    LessEquals("LessEquals"),
    /**
     * 字符串包含
     */
    Like("Like"),
    /**
     * 值包含
     */
    In("In"),
    /**
     * IS NULL
     */
    IsNull("IsNull"),
    /**
     * NOT IS NULL
     */
    IsNotNull("IsNotNull"),
    /**
     * Start With
     */
    Start("Start"),
    /**
     * End With
     */
    End("End"),
    /**
     * Field is empty
     */
    IsEmpty("IsEmpty"),
    /**
     * Field is not empty
     */
    IsNotEmpty("IsNotEmpty");
}
```

#### Equals

`Equals`指定等于查询, `type`的值为实体字段名, `value`的值为需要判断的值,
```json
{
	"conditions": [
		{ "type": "field", "value": "value1" }
	]
}
```
此请求参数生成的sql为`SELECT * FROM TABLE WHERE field = 'value1'`

#### NotEquals

`NotEquals`指定不等于查询, `type`的值为`neq_`+实体字段名, `value`的值为需要判断的值,
```json
{
	"conditions": [
		{ "type": "neq_field", "value": "value1" }
	]
}
```
此请求参数生成的sql为`SELECT * FROM TABLE WHERE field != 'value1'`

#### Great

`Great`指定大于查询, `type`的值为`gt_`+实体字段名, `value`的值为需要判断的值,
```json
{
	"conditions": [
		{ "type": "gt_field", "value": "value1" }
	]
}
```
此请求参数生成的sql为`SELECT * FROM TABLE WHERE field > 'value1'`

#### GreatEquals

`GreatEquals`指定大于等于查询, `type`的值为`gteq_`+实体字段名, `value`的值为需要判断的值,
```json
{
	"conditions": [
		{ "type": "gteq_field", "value": "value1" }
	]
}
```
此请求参数生成的sql为`SELECT * FROM TABLE WHERE field >= 'value1'`

#### Less

`Less`指定小于查询, `type`的值为`lt_`+实体字段名, `value`的值为需要判断的值,
```json
{
	"conditions": [
		{ "type": "lt_field", "value": "value1" }
	]
}
```
此请求参数生成的sql为`SELECT * FROM TABLE WHERE field < 'value1'`

#### LessEquals

`LessEquals`指定小于等于查询, `type`的值为`lteq_`+实体字段名, `value`的值为需要判断的值,
```json
{
	"conditions": [
		{ "type": "lteq_field", "value": "value1" }
	]
}
```
此请求参数生成的sql为`SELECT * FROM TABLE WHERE field <= 'value1'`

#### Like

`Like`指定模糊查询, `type`的值为`like_`+实体字段名, `value`的值为需要判断的值,
```json
{
	"conditions": [
		{ "type": "like_field", "value": "value1" }
	]
}
```
此请求参数生成的sql为`SELECT * FROM TABLE WHERE field LIKE '%value1%'`

#### In

`In`指定IN查询, `type`的值为`in_`+实体字段名, `value`的值为需要判断的值,
```json
{
	"conditions": [
		{ "type": "in_field", "value": "value1" },
        { "type": "in_field", "value": "value2" }
	]
}
```
此请求参数生成的sql为`SELECT * FROM TABLE WHERE field IN ('value1', 'value2')`

#### IsNull

`IsNull`指定ISNULL查询, `type`的值为`null_`+实体字段名,
```json
{
	"conditions": [
		{ "type": "null_field" }
	]
}
```
此请求参数生成的sql为`SELECT * FROM TABLE WHERE ISNULL(field)`

#### IsNotNull

`IsNotNull`指定ISNULL反查询, `type`的值为`notnull_`+实体字段名,
```json
{
	"conditions": [
		{ "type": "notnull_field" }
	]
}
```
此请求参数生成的sql为`SELECT * FROM TABLE WHERE !ISNULL(field)`

#### Start

`Start`指定右模糊查询, `type`的值为`start_`+实体字段名, `value`的值为需要判断的值,
```json
{
	"conditions": [
		{ "type": "start_field", "value": "value1" }
	]
}
```
此请求参数生成的sql为`SELECT * FROM TABLE WHERE field like 'value1%'`

#### End

`Start`指定左模糊查询, `type`的值为`end_`+实体字段名, `value`的值为需要判断的值,
```json
{
	"conditions": [
		{ "type": "end_field", "value": "value1" }
	]
}
```
此请求参数生成的sql为`SELECT * FROM TABLE WHERE field like '%value1'`

#### IsEmpty

`IsEmpty`指定空查询, `type`的值为`empty_`+实体字段名,
```json
{
	"conditions": [
		{ "type": "empty_field" }
	]
}
```
此请求参数生成的sql为`SELECT * FROM TABLE WHERE field = ''`

#### IsNotEmpty

`IsNotEmpty`指定非空查询, `type`的值为`notempty_`+实体字段名,
```json
{
	"conditions": [
		{ "type": "notempty_field" }
	]
}
```
此请求参数生成的sql为`SELECT * FROM TABLE WHERE field != ''`

### 查询字段或者查询类型说明

#### 查询字段

如需指定查询字段, `type`的值为`select_`+实体字段名,
```json
{
	"conditions": [
		{ "type": "select_field" }
	]
}
```
此请求参数将只查询指定的字段, `SELECT field FROM TABLE ...`

#### 查询平均值

如需查询平均值, `type`的值为`avg_`+实体字段名,
```json
{
	"conditions": [
		{ "type": "avg_field" }
	]
}
```
此请求参数查询指定的字段的平均值, `SELECT AVG(field) FROM TABLE ...`<br>
<label style='color:red'>注意</label>: 查询字段需为数字类型

#### 查询数量

如需查询记录数, `type`的值为`count_`+实体字段名,
```json
{
	"conditions": [
		{ "type": "count_field" }
	]
}
```
此请求参数查询记录数, `SELECT COUNT(*) FROM TABLE ...`

#### 查询最小值

如需查询最小值, `type`的值为`min_`+实体字段名,
```json
{
	"conditions": [
		{ "type": "min_field" }
	]
}
```
此请求参数查询指定字段的最小值, `SELECT MIN(field) FROM TABLE ...`<br>
<label style='color:red'>注意</label>: 查询字段需为数字类型

#### 查询最大值

如需查询最大值, `type`的值为`max_`+实体字段名,
```json
{
	"conditions": [
		{ "type": "max_field" }
	]
}
```
此请求参数查询指定字段的最大值, `SELECT MAX(field) FROM TABLE ...`<br>
<label style='color:red'>注意</label>: 查询字段需为数字类型

#### 查询合计值

如需查询合计值, `type`的值为`sum_`+实体字段名,
```json
{
	"conditions": [
		{ "type": "sum_field" }
	]
}
```
此请求参数查询指定字段的合计值, `SELECT SUM(field) FROM TABLE ...`<br>
<label style='color:red'>注意</label>: 查询字段需为数字类型

#### 指定GROUP BY

如需指定GROUP BY, `type`的值为`group_`+实体字段名,
```json
{
	"conditions": [
		{ "type": "group_field" }
	]
}
```
此请求参数指定GROUP BY字段, `SELECT ... FROM TABLE ... GROUP BY field`

#### 升序查询

如需指定升序查询, `type`的值为`asc_`+实体字段名,
```json
{
	"conditions": [
		{ "type": "asc_field" }
	]
}
```
此请求参数指定升序查询, `SELECT ... FROM TABLE ... ORDER BY field ASC`

#### 倒序查询

如需指定倒序查询, `type`的值为`desc_`+实体字段名,
```json
{
	"conditions": [
		{ "type": "desc_field" }
	]
}
```
此请求参数指定倒叙查询, `SELECT ... FROM TABLE ... ORDER BY field DESC`



接受`{ "type": "${field}", "value": "${value}" }`格式的条件列表, 可根据业务需求自由组合查询条件