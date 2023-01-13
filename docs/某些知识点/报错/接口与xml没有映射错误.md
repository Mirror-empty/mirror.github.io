# 接口与xml没有映射错误

错误

```java
org.apache.ibatis.binding.BindingException: Invalid bound statement (not found):
```

这是mybatis中的dao接口与mapper配置文件没有做映射绑定

用mybatis-x插件可以跳过去，但还是启动报错因为，target文件中没有编译
启动一下application启动类就行，然后在到test中测试