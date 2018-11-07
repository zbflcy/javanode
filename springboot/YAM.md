# YAML 语法
 
 ## 基本语法
k:(空格)v： 表示一对键值对，（空格必须有）  
以空格的缩进表示层级关系；只要是左对齐的一列数据，都是一个层级的。
```yaml
    server:
        port: 8801
        path: /hello
```
属性和值大小写敏感的。 
## 值的写法
### 字面量 ：普通的值（数字，字符串，布尔）
K: V 直接写  
并且字符串默认不用加上单引号或者双引号。“”双引号不会转义字符串里面的特殊字符。‘’会转义特殊字符。
### 对象、map（属性和值）（键值对）
1. 行间写法  
在下一行写对象的属性和值；注意缩进，对象还是K: V的方式
```yaml
    friends:
        lastName: zhangshan
        age: 20
```
2. 行内写法  
```yaml
    friends: {lastName: zhangshan,age: 18}
```
### 数组
1. 行间写法  
用-（空格）值表示数组中的一个元素
```yaml
    pets:
        - cat
        - dog
        - pig
```

2. 行内写法  
```yaml
    pets: [cat,dog,pig]
```


