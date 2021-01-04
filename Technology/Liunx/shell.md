### bash 快捷键

[Bash 快捷键大全 ](https://linux.cn/article-5660-1.html)

| 键            | 说明                                         |
| ------------- | -------------------------------------------- |
| ctrl - a      | 将光标移到行首                               |
| ctrl - e      | 将光标移动到行尾                             |
| ctrl - u      | 删除从光标所在处到行首的所有字符             |
| ctrl - k      | 删除从光标所在处到行尾的所有字符             |
| ctrl - w      | 会删除从在光标处往前第一个空白符之间的内容。 |
| alt shift - b | 删除光标后第一个空白符之间的内容。           |
| ctrl - /      | 撤消操作, Undo                               |
| ctrl - r      | 搜索输入过的命令                             |
| ctrl 方向键   | 每次移动一个单词                             |

### shell 类型

- sh (Bourne shell)

- bash (Bourne shell)

- ksh (korn shell)

- csh (c shell)

- dash (Debian Almquist shell)

  ```bash
  # 查看sh类型
  ll /bin/sh
  ```

  

### 注释

```bash
#1.单行注释 
行前加#
#2.多行注释
:<<EOF
this is a annotation;
EOF
```

### 变量

`shell是动态语言，不需要类型限定`

##### 1. string

```bash
str="string"   # str = "string" 是相等操作
num=123  # s="123"

#打印
echo $str ${str}
echo $num + $str

# 字符串长度 　
echo ${#str}

#获取文件名和文件后缀
file=somefile.txt.tar.gz
echo ${file#*.}
echo ${file%%.*}  # 两个%表示贪婪模式

#切片
str=abcdefg
echo ${str:2:3}  # cde
#替换
echo ${str/ab/xx} # xxcdefg  ${str//ab/xx} 双/表示贪婪模式

# 格式化字符串 printf打印
printf  "%s %d %0.2f\n"  "str" 3 2.342
```

##### 2. array

```bash
arr=(n1 n2 n3)
arr=([0]=n1 [1]=n2 [3]=n3)
# 打印
echo $arr  # 等于${arr[0]}
echo ${arr[0]}
echo ${arr[@]}  # 打印全部
echo ${#arr}    # 打印长度
echo ${!array_var[*]} # 列出数组索引

```
**变量声明 declare**

```bash
declare -i i
i=somestr
echo $int # 输出 0, i的值非int类型都输出为0
```

**取消变量**

```bash
unset var
```

#### 变量作用域

​    全局变量、局部变量与环境变量

​    

​	 **子shell** (在shell脚本中执行shell脚本)

​     子shell是看不到父shell的变量

​     **export 命令用于设置或显示环境变量。**

```bash
#  默认是全局变量
local var=abc   # 限定为局部变量
export var=abc   #　变量提升为环境变量，在子shell中可以使用var这个变量
```

### 执行子命令

1. $(cmd)   # 建议用

2. \`cmd\`

   ```bash
   #获取3-10中的数
   echo $(seq 3 10)
   echo `seq 3 10`
   ```

### 运算符

##### 1.算术运算

```bash
# 1.let 
let n3=n1+n2
let n1++
let n1+=4
# 2.[]
n3=$[n1+n2]  #n1,n2加不加$都行
# 3.(())  # 还支持逻辑运算
n3=$((n1+n2))
((n3=n1+n2))


```

##### 2.关系运算

| [  ] | [[ ]] |      |
| :--: | :---: | ---- |
| -eq  |  ==   |      |
| -ne  |  !=   |      |
| -gt  |   >   |      |
| -lt  |   <   |      |
| -ge  |       |      |
| -le  |       |      |

```bash
#[ ] 
[ 3 -eq 3 ]
[[ 3 > 3 ]]
```

##### 3.逻辑运算

| [ ]  | [[ ]] |      |
| :--: | :---: | ---- |
|  -a  |  &&   | 与   |
|  -o  | \|\|  | 或   |

```bash
[ $n -eq 2 -a $r -eq 3 ]
[[ $n -eq 2 && $r -eq 3 ]]
```

### test

```bash
# 使用
test -n $var
[ -n $var ]
```



#### 字符串测试

| 参数      | 说明                                             |
| --------- | ------------------------------------------------ |
| =         | 等于则为真                                       |
| !=        | 不相等则为真                                     |
| -z 字符串 | 字符串的长度为零则为真                           |
| -n 字符串 | 字符串的长度不为零则为真, -n "$var" , 一定要引号 |

#### 文件测试

| 参数      | 说明                                 |
| --------- | ------------------------------------ |
| -e 文件名 | 如果文件存在则为真                   |
| -r 文件名 | 如果文件存在且可读则为真             |
| -w 文件名 | 如果文件存在且可写则为真             |
| -x 文件名 | 如果文件存在且可执行则为真           |
| -s 文件名 | 如果文件存在且至少有一个字符则为真   |
| -d 文件名 | 如果文件存在且为目录则为真           |
| -f 文件名 | 如果文件存在且为普通文件则为真       |
| -c 文件名 | 如果文件存在且为字符型特殊文件则为真 |
| -b 文件名 | 如果文件存在且为块特殊文件则为真     |

### 函数

```bash
# 定义函数
function func_name (){   # function可有可无
     # 获取参数
     echo $1 $2
}
# 调用
func_name arg1 arg2 # 函数名后为函数参数
```

### 流程控制 

##### 1. if 

```bash
num=3
if [ $num -lt 3 ]
then
	echo "the num is less than 3"
elif [ $num -gt 3]
then
	echo "the num is greater than 3"
else
 	echo "the num is equal to 3"
fi
# 写成一行
if [ num<3 ];then ... ;fi  # ";"等同于换行
```



##### 2. for

```bash
for var in item1 item2 ... itemN
do
    ...
done
# C语言风格
for ((i=0;i<10;i++))
do
	...
done
```

#####  3. while

```bash
while condition
do
    command
done
```

##### 4. case

```bash
site="google"
case "$site" in
   "google") echo "Google 搜索"
   ;;   # 一定要有;;, 不然会继续执行后面的
   "taobao") echo "淘宝网"
   ;;
esac
```

### IFS  

> ​    IFS（Internal Filed Separator，内部域分隔符）是一种 set 变量，当 shell 处理字符串时，shell会根据 IFS 的值，默认是空格、Tab键、换行来解析读入的变量，然后对特殊字符进行处理，最后重新组合赋值给该变量。
>

[Shell中的IFS解惑](https://blog.csdn.net/whuslei/article/details/7187639)

```bash
# 查看IFS的值
echo "$IFS"|od -b
0000000 040 011 012 012  
0000004
"040"是空格，"011"是Tab，"012"是换行符"\n" 
```
用途

数组a=(a b c "d e"), for循环

**$*与$@**

```bash
echo '--------$*-----'
for i in $*;do
        echo $i
done
echo '--------$@-----'
for i in $@;do
        echo $i
done

echo "加引号后:"

echo '--------"$*"-----'
for i in "$*";do
        echo $i
done
echo '--------"$@"-----'
for i in "$@";do
        echo $i
done
```

![脚本输出](./asterisk.png)

### 登录式shell与非登录式shell

1. **登录shell (ssh远程上去)**

```bash
# 读取配置文件顺序
/etc/profile --> /etc/profile.d/*.sh --> ~/.bash_profile–> ~/.bashrc–> /etc/bashrc
```

2. **非登录shell(ssh执行命令)**

```bash
   ~/.bashrc–> /etc/bashrc–> /etc/profile.d/*.sh
```

### []与[[]]

1. **[]等同与test**  `[ -n "$a" ] 等于 test -n $a `

2. [[  ]] 是bash才支持的，其他shell执行会报错

3. [], [[]] 左右要有空格

4. []要有空值保护，[[]]不需要 `[ $a -eq 1 ] 如果$a等于空，就变成 [ -eq 1 ]语法解析会出错。一般写成[ x$a -eq x1 ]`

5. [[ ]]支持正则，**正则不能加引号**

   ```bash
    # =~ 是包含的关系，如果a包含s开头加数字则打印OK
    if [[ $a =~ ^s[0-9] ]];then echo "OK" ;fi
   ```

### Vim

#### 1. 四种模式

- **正常模式(normal)**

  >  vim打开文件时的默认模式, 无论在哪种模式下，按下Esc键就会进入正常模式。

- **命令模式(command)**

  >  在正常模式下输入“:”或“/”进入命令行模式，在该模式下可以进行保存，搜索，替换，退出，显示行号等

- **插入模式 (insert)**

  > 在正常模式下按下i键，进入插入模式

- **可视模式(visual)**

  > 在正常模式下按v（小写）进入字符文本，按V（大写）进入行文本，按ctrl+v进入块文本。

#### 2. 快捷键

1. 移动光标

   ```
   h ：光标左移一个字符
   j ：光标上移一个字符
   k ：光标下移一个字符
   l ：光标右移一个字符
   
   0 ：光标移至行首
   $ ：光标移至行尾
   
   H ：光标移至屏幕首行
   M ：光标移至屏幕中间
   L ：光标移至屏幕最末行
   
   ng: 3g，移到第三行， gg, 移至第一行。
   G:  光标移到最后一行
   
   w: 向右移一个单词
   b: 向前移一个单词
   ```

2. 插入文本

   ```
   i ：在光标前插内内容
   a ：在光标后插入内容
   o ：在所在行的下一行插入新行
   O ：在所在行的上一行插入新行
   ```

3. 删除文本

   ```
   ndd: dd剪切当前行， 剪切n行
   dw:  删除当前光标至单词末尾
   d0： 删除光标至行首的内容
   d$： 删除光标至行尾的内容
   u:   取消上一次操作
   ctrl + r： 撤消取消
   ```

4. 复制文本

   ```
   nyy: yy复制当前行， 3yy复制当前至第三行
   p:   所在行下一行粘贴
   P:   所在行上一行粘贴
   ```

5. 屏幕翻滚

   ```
   ctrl+u：向文件首翻半屏
   ctrl+d：向文件尾翻半屏
   ctrl+f：向文件尾翻一屏
   ctrl+b：向文件首翻一屏
   ```
   
6. 查找文本

   ```
   /pattern ：向下查找
   ?pattern ：向上查找
   n ：顺序查找
   N ：反向查找
   ```

7. 末行命令

   ```
   :n1,n2d  n1行到n2行删除， d是删除， s是替换
   :n1,n2 co n3  ：将n1至n2行复制到n3行的下面
   ```

#### 3. 正则表达式

默认是magic模式， :set xx可切换成其他模式

\m,\M等是临时开关

|                   |                                         |
| ----------------- | --------------------------------------- |
| magic  (\m)       | 除了 $ . * ^ 之外其他元字符都要加反斜杠 |
| nomagic (\M)      | 除了 $ ^ 之外                           |
| very magic (\v)   | 任何元字符都不用加反斜杠                |
| very nomagic (\V) | 任何元字符都必须加反斜杠                |

##### 元字符

###### (1) 元字符一览

| 元字符     | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| **.**      | 匹配任意一个字符                                             |
| **[abc]**  | 匹配方括号中的任意一个字符。可以使用-表示字符范围， 如**[a-z0-9]**匹 配小写字母和阿拉伯数字。 |
| **[^abc]** | 在方括号内开头使用**^**符号，表示匹配除方括号中字符之外的任意字符。 |
| **\d**     | 匹配阿拉伯数字，等同于**[0-9]**。                            |
| **\D**     | 匹配阿拉伯数字之外的任意字符，等同于**[^0-9]**。             |
| **\w**     | 匹配单词字母，等同于**[0-9A-Za-z_]**。                       |
| **\W**     | 匹配单词字母之外的任意字符，等同于**[^0-9A-Za-z_]**。        |
| **\t**     | 匹配<TAB>字符。                                              |

###### (2) 表示数量的元字符

| 元字符     | 说明         |
| ---------- | ------------ |
| *****      | 匹配0-任意个 |
| **\+**     | 匹配1-任意个 |
| **\?**     | 匹配0-1个    |
| **\{n,m}** | 匹配n-m个    |
| **\{n}**   | 匹配n个      |
| **\{n,}**  | 匹配n-任意个 |
| **\{,m}**  | 匹配0-m个    |

###### (3) 表示位置的符号

| 元字符 | 说明         |
| ------ | ------------ |
| **$**  | 匹配行尾     |
| **^**  | 匹配行首     |
| **\<** | 匹配单词词首 |
| **\>** | 匹配单词词尾 |

### Sed 

> Sed是Stream EDitor的缩写, sed是行编辑，awk是列编辑

**正则和vim 的magic模式差不多**

```bash
sed 's/\([a-z]\{3\}\) i/\1 b/g' /etc/profile
# /随便写成什么字符, @, # 都行。 sed 's#abc#bcd#g' xx
```

##### (1) 定址

```bash
sed -n '3p' datafile
#只打印第三行

# 只查看文件的第100行到第200行
sed -n '100,200p' mysql_slow_query.log

sed '2,5d' datafile
#删除第二到第五行
sed '/My/,/You/d' datafile
#删除包含"My"的行到包含"You"的行之间的行
sed '/My/,10d' datafile
#删除包含"My"的行到第十行的内容
```

##### (2)  sed命令

| 命令 | 功能                                                       |
| ---- | ---------------------------------------------------------- |
| a    | 在当前行后添加一行或多行。sed "/unset i/aabc" /etc/profile |
| c    | 用此符号后的新文本替换当前行中的文本。                     |
| i    | 在当前行之前插入文本。                                     |
| d    | 删除行                                                     |
| p    | 打印行                                                     |
| s    | 用一个字符串替换另一个                                     |
| g    | 在行内进行全局替换                                         |

##### (3) sed选项

| 选项 | 功能                                                         |
| ---- | ------------------------------------------------------------ |
| -e   | 进行多项编辑，多条sed命令时使用， sed -e '1,10d' -e 's/My/Your/g' datafile |
| -i   | 替换原文件                                                   |
| -n   | 取消默认的输出                                               |
| -f   | 指定sed脚本的文件名                                          |

### awk 

> AWK 是一种处理文本文件的语言，是一个强大的文本分析工具。
>
> 之所以叫 AWK 是因为其取了三位创始人 Alfred Aho，Peter Weinberger, 和 Brian Kernighan 的 Family Name 的首字符。

#### 语法

```bash
awk [选项参数] 'script' var=value file(s)
选项参数说明：
    -F fs or --field-separator fs
        指定输入文件折分隔符，fs是一个字符串或者是一个正则表达式，如-F:。 
        eg: awk -F: '{print $1 $2}' score.txt
```

#### 内建变量

| 变量     | 描述                                                       |
| -------- | ---------------------------------------------------------- |
| $n       | 当前记录的第n个字段，字段间由FS分隔                        |
| $0       | 完整的输入记录                                             |
| FILENAME | 当前文件名                                                 |
| FNR      | 各文件分别计数的行号                                       |
| FS       | 字段分隔符(默认是任何空格)                                 |
| NF       | 一条记录的字段的数目                                       |
| NR       | 已经读出的记录数，就是行号，从1开始                        |
| OFS      | 输出字段分隔符（输出换行符），输出时用指定的符号代替换行符 |
| ORS      | 输出记录分隔符(默认值是一个换行符)                         |
| RS       | 记录分隔符(默认是一个换行符)                               |

```bash
awk 'BEGIN{OFS=":";ORS="&"}{print $1, $3+s}' s=5 score.txt
# 打印第三列小于60分的同学
cat score.txt  | awk '$3<60{print $1,$3}'
cat score.txt  | awk '{if($3<60){print $1,$3}}'  #  语言类似c语言

# 打印所有列
awk '{print $0}' score.txt
# 打印除第一行的所有列
awk '{$1=""; print $0}' score.txt
# 打印最后一列
awk '{print $NF }' score.txt
# 打印序号
awk '{print NR, $0}' score.txt
```

#### awk脚本

关于 awk 脚本，我们需要注意两个关键词 BEGIN 和 END。

- BEGIN{ 这里面放的是执行前的语句 }

- END {这里面放的是处理完所有的行后要执行的语句 }

- {这里面放的是处理每一行时要执行的语句}

```bash
[root@localhost ~]# cat a.awk
{
    if($3<60){
        print $1,$3
    }
}
[root@localhost ~]# cat score.txt  | awk -f a.awk
```



