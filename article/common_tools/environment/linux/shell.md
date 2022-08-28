# shell基本

+ 查看当前使用的shell：`echo $SHELL`
+ 查看本系统已安装的所有shell：`cat /etc/shells`
+ 查看 版本：`bash --version`

**命令组合：**

+ `Command1 && Command2`，1成功才执行2
+ `Command1 || Command2`，1失败才执行2
+ `Command1 ; Command2`，1执行完执行2

**快捷键：**

+ `Ctrl + L`：清除屏幕
+ `Ctrl + C`：中止
+ `Ctrl + D`：关闭会话
+ `Ctrl + U`：从光标位置删除到行首
+ `Ctrl + K`：从光标位置删除到行尾

**字符匹配：**

+ 匹配文件名单字符：`?`
+ 匹配文件名任意字符：`*`
+ 文件名列举匹配：
  + ` [ab]`、`[!abc]`、`[a-zA-Z]`
  + `{1,2,3}`、`{j{p,pe}g,png}`、`{a..g}`、`{0..8..2}`(2步长)、`{2007..2009}{01..05}`(循环3*5)

**变量匹配：**

+ 精准匹配：`$SHELL`、`${SHELL}`
+ 模糊匹配：`${!关键字*}`

**命令匹配：**`$(date)`

**输出重定向：**

+ 覆盖：`command > file`、`fileA > fileB`
+ 追加：`command >> file`、`fileA >> fileB`

**输入重定向：**

+ 覆盖：`command < file`



# 变量

**定义变量：**

+ `variable="value"`，此时variable将直接引用value的字符串内容
+ `variable=$(order)`，此时variable将间接引用order的执行内容
+ `variable=$((2*3))`，此时variable将间接引用 (2*3) 的计算结果
+ `export variable="value"`，定义全局变量

**使用变量：**

+ `${variable}`，获取制式样式的结果
+ `"${variable}"`，获取自定义的样式结果

**删除变量：**

+ `unset variable`

# 脚本

**须知：**

+ 第一行必须加解释器，声明用哪种执行器执行此文件，如 `#!/bin/bash`
+ 执行方式：
  + `./*.sh` 是以脚本的方式执行文件内的命令，需要权限
  + `sh *.sh` 是以解释器运行脚本，不需要权限



**引入文件：**

+ `source file`



