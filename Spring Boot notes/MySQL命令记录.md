### 用户权限相关

创建用户

```
CREATE USER 'username'@'%' IDENTIFIED BY 'password'; 
```

授权

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456';
flush privileges;
```

这里的123456为你给新增权限用户设置的密码，%代表所有主机，也可以具体到你的主机ip地址

flush privileges; 这一步一定要做，不然无法成功！ 这句表示从mysql数据库的grant表中重新加载权限数据。因为MySQL把权限都放在了cache中，所以在做完更改后需要重新加载。

