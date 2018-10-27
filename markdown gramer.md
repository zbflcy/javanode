# markdown learn node

#### comment{#comment}

&ensp;&ensp;markdown沿用html的注释风格使用&lt;!-- 内容 -->来进行注释  
* 快捷键：ctrl+/

#### 2、字体
代码：
```
  1. *斜体* 或者 _斜体_
  2. **粗体**
  3. *** 加粗斜体***
  4. ~~删除线~~
```
显示效果：
* *这是一段斜体*
* **这是一段加粗**
* ***这是一段加粗斜体***
* ~~这是一段删除线~~

#### 标题 3、header
两种写法
```
第一种写法
    这是一级标题
    ===
    这是二级标题
```
```
第二种写法
  # 一级标题
  ## 二级标题
  ### 三级标题
  #### 四级标题
  ##### 五级标题
  ###### 六级标题
```

#### 4、超链接
markdown支持两种超链接语法：行内式和参考式两种形式。
##### 4.1 行内式
语法说明：
* []里面是链接文字，()里面是链接地址，[链接文字]\(链接地址)，  
  同时也可以在()内指定title,就是鼠标悬停时显示的文字。[]\(链接地址，title）。

代码：
```
1. 欢迎来到[百度](http://www.baidu.com)
2. 欢迎来到[百度](http://www.baidu.com "百度") 注意中间是空格
```
效果：
1. 欢迎来到[百度](http://www.baidu.com)
2. 欢迎来到[百度](http://www.baidu.com "百度")

##### 4.2参考式""
参考式链接适用于一个链接在文章内多处使用的情况，方便统一管理链接  
语法说明：
```
参考式链接分为两个部分，在正文中的写法是[链接文字][链接标记]，
此外，如果链接文字本身就是链接标记，则可以写成[链接文字][]
在文中的任意位置添加链接标记，格式：[链接标记]:链接地址"链接标题
```
代码：
```
我经常去的几个网站[虎扑][1]、[百度][2]、[知乎][3]、[谷歌][]

[1]:http://www.hupu.com "hupu"
[2]:http://www.baidu.com "baidu"
[3]:https://zhihu.com "知乎"
[谷歌]:https://www.google.com.hk "google"
```
效果：  
我经常去的几个网站[虎扑][1]、[百度][2]、[知乎][3]、[谷歌][]  

[1]: https://hupu.com "hupu"
[2]: http://www.baidu.com "baidu"
[3]: htts://zhihu.com "zhihu"
[谷歌]: https://www.google.com.hk

#### 4.3自动链接
通过<>，markdown都可以自动把它转换成链接，
代码：
```
  <https:google.com>
  <https:zhihu.com>
```
效果:  
1. <https://google.com>
2. <https://zhihu.com>


back to [comment](#comment)
