# 正则表达式

```java
String text = "中央军委主席习近平日前签署通令，给1名个人、1个单位记功。" +
				"给中国人民解放军92853部队部队长孙宝嵩记战备训练一等功，给中国" +
				"人民解放军96763部队记二等功。";
		String reg = "\\d{5}";
		Pattern pattern = Pattern.compile(reg);
		Matcher matcher = pattern.matcher(text);
		while (matcher.find()) {
			System.out.println(matcher.group(0));
		}
```

## 原理：

1. `Pattern.compile(reg)` 根据给定的字符串创建一个正则对象

2. `pattern.matcher(text)` 创建一个根据指定正则对象匹配目标文本的匹配器

   ![image-20220728000515806](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220728000515806.png)

3. `matcher.find()` 根据正则规范查找符合的子字符串，并将当前匹配的结果索引保存到groups数组中。该方法只会存储当前查找匹配的索引，即会循环覆盖。当

   ![image-20220728001317733](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220728001317733.png)

   ![image-20220728001620379](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220728001620379.png)

4. `matcher.group(0)` , 取出整体匹配规范得到的结果，同理 `matcher.group(i)`  为取出第 i 个分组子规范的匹配结果。分组由匹配规则 reg中的"()" 指定，在创建正则对象时声明。

   ```
   // 非命名分组 ，后续取的时候只能按编号取，即 matcher.group(0)
   String reg = "([a-z]+)(//d{2})";
   
   // 命名分组 ，后续取的时候还能按组名取，即 matcher.group("zm")
   String reg = "?<zm>([a-z]+)(?<sz>//d{2})";
   ```

 

## 正则语法

1. 

### 范围字符

| 字符          | 匹配                                                        |                                               |
| ------------- | :---------------------------------------------------------- | --------------------------------------------- |
| `.`任意字符   | 除**换行符**外的任意单个字符                                | 添加`s`修饰符，可以让`.`包括换行符 即`(\s|.)` |
| `\w` 单词字符 | 字母、数字、下划线任意单个字符。大写`\W` 表示**非单词**字符 | 在python `\w`还可以表示汉字                   |
| `\d` 数字     | 0-9任意单个数字。大写`\D` 表示**非数字**                    |                                               |
| `\s` 空白字符 | 空格、tab制表符、换行都属于空白字符。大写`\S` 非空白字符    |                                               |

### 逻辑符

| 逻辑符        | 描述                                                         | 兼容性 |
| :------------ | :----------------------------------------------------------- | :----- |
| `|` 或        | 如`hi|hello|hellow` 匹配其中一个单词即可。                   |        |
| `()` 子表达式 | 用于独立计算括号中的内容。如：`(张|李)建国` 表示第一个字符是张**或者**李。 |        |
| `{}` 数量控制 | 对字符或字表达式的数量范围进行限定。如：`张.{1,3}` 表示“张”后面只能跟**1到3**个任意字符。 |        |

<u>注意：无括号和圆括号内默认是&&，[ ] 内默认是 ||。</u>

### 数量符

| 逻辑符           | 描述                                         | 兼容性 |
| :--------------- | :------------------------------------------- | :----- |
| `{n}` 指定数量   | 匹配**n**个字符                              |        |
| `{n,m}` 指定范围 | 匹配**n**至**m**个字符，即至少n个，最多m个。 |        |
| `{n,}` n个及以上 | 匹配**n**个或者n个**以上**字符               |        |
| `*` 任意个       | 相当于`{0,}` 0个或无数个                     |        |
| `+` 至少1个      | 相当于 `{1,}`                                |        |
| `？` 0或1个      | 相当于 `{0,1}`                               |        |

### 惰性匹配

​	`.*?` 。`?`表示最小化匹配，也叫懒惰匹配，只能用在量词后面，表示如果多个文本段同时满足条件，则匹配最短的。`?`可以用在所有量词后面，甚至是`??` ,但这个量词不能是固定的值，如:`hel{2}?o` 这是没有意义的。因为它不存在最少和最多。

### 大小写转化

【**注意**】：该符在匹配过程中不起效，只在修改，替换过程中生效。

| 操作符             | 描述                           | 兼容性 |
| :----------------- | :----------------------------- | :----- |
| \u 单个转大写      | 转换一下个字符为**大**写       |        |
| \U 全部转大写      | 转换`\U`后所有字符转**大**写   |        |
| \U...\E 区间转大写 | `\U`与`\E`区间的内容转**大**写 |        |
| \l 单个转小写      | 转换一下个字符为小写           |        |
| \L 全部转小写      | 转换`\L`后所有字符转小写       |        |
| \L...\E 区间转小写 | `\L`与`\U`区间的内容转小写     |        |

### 分组

1. `(?<名称> )`命名分组，可使用组名获取匹配内容
2. `(?: )` 移除分组，即无法存储该分组所匹配的内容，无对应的分组索引
3. `( ( ) )`嵌套分组，组号的命名顺序是以**左开括号**出现顺序为准
4. `()+`分组使用量词，这时通过`$组号`去提取值的时候会得到该组最后匹配的值。如`(\d)+` 匹配12345，通过`$1`将得到5



### 反向引用

将匹配的结果作为参数，继续匹配，必需要采用分组。语法：`(g1)(g2)\g1\g2`【\g1 意为引用分组1】，如匹配 三连数"111",`([0-9]\1{2})`，匹配回串1221：`([0-9][0-9]\2\1)`。

**注意：** 在 regStr 中反向引用只能通过`\组号`进行，且只能引用表达式前面定义的组（不能出现在目标组之前），不能引用被`(?:)`移除的组

**注意：**如果要**在 regStr 之外反向引用当前的匹配结果**，需要使用`$组id` 或者`$组名`，如`matcher.replaceAll("$1");`

```java
		String text1 = "5112334";
		String reg = "([0-9])\\1";
		Pattern pattern = Pattern.compile(reg);
		Matcher matcher = pattern.matcher(text1);
		String s1 = matcher.replaceAll("$1");
```



## String直接使用正则

```java
// 直接替换
String ok = text1.replaceAll("([0-9])\\1", "$1");

// 验证匹配
text.matches(regex) 
  
// 字符串分割
text.split(regex)
```

