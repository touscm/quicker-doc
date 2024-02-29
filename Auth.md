# 认证

认证采用Bearer Token协议进行认证, 认证参数有group和token, group用来定义组织, token通常来自登陆返回结果, 具体格式为
`{ "group": "user-auth-group", "token": "login-res-token" }`, 然后使用Base64对该对象进行编码, 使用"Bearer "加上编码后的内容就是认证头内容, 例如
```text
Bearer eyJncm91cCI6IlBsYXRmb3JtIiwidG9rZW4iOiI0OWJlNzI1ZGY0NzM0YzM3OTIzYzQzZmQ3ODM4MmQzMSJ9
```
把认证内容放到"Authorization" Http里就可以访问接口, curl示例如下
```shell
curl -X POST -H 'Content-Type:application/json' \
  -H 'Authorization: Bearer eyJncm91cCI6IlBsYXRmb3JtIiwidG9rZW4iOiI0OWJlNzI1ZGY0NzM0YzM3OTIzYzQzZmQ3ODM4MmQzMSJ9' \
  -d '{"model":{"name":"12ec810838464209b9ede2f096dee935"}}' \
  http://127.0.0.1:80/api/sysRoleMenu/add
```
