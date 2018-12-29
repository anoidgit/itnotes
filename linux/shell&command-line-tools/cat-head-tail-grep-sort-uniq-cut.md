一些Linux常用的文本处理工具。

---

[TOC]

# cat/more/less/tail/head

## cat打印文件内容

`cat 选项 文件名`

- 显示行号

  - `-n`或`--number`  显示每行编号
  - `-b`或 `--number-nonblank`空白行不进行编号
  - `-s`或`--squeeze-blank`  续两行以上的空白行显示为一行

- 显示特殊字符

  `-A`或`--show-all`等价`-vET`；`-e`等价`-vE`；`-t`等价`-vT` 。

  - `-v`或`--show-nonprinting`  使用`^`和`M-`符号（除`LFD`和`TAB`）
  - `-E`  `--show-ends  `  在行尾显示`$`符号（即在行尾显示`$` ）
  - `-T`  `--show-tabs`  将tab字符显示为`^|`符号

示例：

```shell
cat /etc/enviroment -n -E  #查看文件显示行号和行尾$符号
cat /dev/null > file  #清空文件内容可以将/dev/null写入到指定文件中
```

### tac和rev

`cat`是正序按行打印文件内容，而`tac`（cat反过来）倒序按行打印文件内容。

`rev`每行内容倒序打印。

## more和less阅读文件内容

`cat`读取文件时会全部打印出所有内容的，而`more`和`less`则进入浏览器模式，在阅读长文档时比cat更有优势，支持上下翻页，`less`比`more`功能更多一些，可使用`--hlep`参数查看使用帮助。阅读模式下常用按键：

- `space`  下翻页
- `b`  上翻页
- `q`  退出

## head显示文件头部内容

显示文件的头部内容，默认显示的开始10行。	`head 选项 文件名`

- `-n`或`--lines=`  指定显示行数
- `-c`或`--bytes=`  指定显示字节数

```shell
head -n 20 file  #显示头部20行的内容
head -c 120 file  #显示头部120字节的内容
```

## tail显示文件尾部内容

显示文件的尾部内容，默认显示的末尾10行。`tail 选项 文件名`

选项：

- `-n`或`--lines=`  指定显示行数   如果在数字n前使用加号`+`，表示第n行到末尾的行

- `-c`或`--bytes=`  指定显示字节数

- `-f`或`--follow`  显示文件最新追加的内容

  参数后面指定监视方式name或descriptor

```shell
tail file    #显示file文件最后10行
tail +20 file    #显示file文件第20行至文件末尾的内容
tail -c 10 file    #显示file文件最后10个字符
```

# grep提取行

`grep 选项 匹配的内容 文件名`  按指定匹配格式提取并显示含有匹配内容的行

## grep常用选项

- `-v`或`--revert-match` **反向**选择（**除匹配项所在行之外的行**，相当于过滤功能）grep使用`-V`查看版本号
- `-i`或`--ignore-case` 忽略大小写
- `-n`或`--line-number `  显示行号
- `-o`或`--only-matching`   只显示匹配的部分
- `-c`或`--count`  找到匹配内容的次数
- `-a`或`--text`  将binary（二进制）文件以text（文本）文件的方式搜寻数据
- `-l`或`-file-with-matches`  从多个文件中查找匹配项
- `-f`或`--file=FILE`  从指定文件中取得匹配规则
- `-r`或`--recursive `  在文件夹中递归查找
- `-q`或 `-quiet`或`--silent` 不显示匹配结果而是输出退出码（0表示匹配成功）
- `--color=auto`  将找到的关键词部分加上颜色的显示

### 匹配

- **在使用grep进行匹配内容时，推荐使用引号将匹配规则包含起来。**（参看下文[元字符](#元字符)中特殊字符的相关说明）
- 正则匹配时最好使用`-P`或`-E`（参看下文[#匹配模式](匹配模式)中相关说明），表示数字使用`[0-9]`而不是`\d`（参看下文[元字符](#元字符)中特殊字符的相关说明）。

#### 匹配模式

- `-e`或`--regexp=PATTERN`  指定字符串作为匹配规则
- `-G`或`--basic-regexp`  基本正则（**默认**）
- `-E`或`--extended-regexp`  扩展正则（同`egrep` ）
- `-P`或`--perl-regexp`   perl正则引擎
- `-F`或`--fixed-strings`  不使用正则（按照字面意思匹配）
- `-w`或`--word-regexp`   仅完全匹配的字词
- `-x`或`--line-regexp`  仅完全匹配一行


`-P`和`-E`和`G`只能选用一个，默认为`-G`（即使不写出该选项）；`grep -E`相当于`egrep`，二者可互相替换。

#### 元字符

这里不叙述正则表达式的元字符内容，而是列出在`grep`中使用元字符的一些注意事项：

- 特殊字符**本身**

  - `.` `\` `$` `^` `[]` `*` `?` `|`等需要使用`\`转义 
  - 单独表示一些符号也可只使用引号而不使用`\`转义（**不推荐**）：`*` `?` `|`
  - 单独表示一些符号不仅要使用`\`转义还必须使用引号：`\` `^` `$`

  为避免使用混乱，**表示特殊字符时一律推荐使用转义符并加上引号。**

- 特定字符类别

  - `\d`和`\D`需要使用`-P`选项，才能被识别出表示数字和非数字类别的特殊意义

    为避免未使用`-P`而造成的匹配失败，**一律推荐使用`[0-9]`和`[^0-9]`代替`\d`和`\D` 。**


示例：

```shell
grep 'Hz'  /proc/cpuinfo  #显示cpu信息
ip addr | grep -o -E '1[^2][0-9?](\.[0-9]{1,3}){3}\/' -o | grep -o -E '1[^2][0-9?](\.[0-9]{1,3}){3}' --color  #本机内网ip
grep -o -E '\b[0-9]{1,3}(\.[0-9]{1,3}){3}\b' /etc/resolv.conf  #查看DNS服务器的IP地址
```

# column格式化输出

常用参数：

- `-t`  将内容创建为表格形式显示
- `-c`  指定列宽（一列的字符数量）
- `-s`  指定分隔符（默认为空格）
- `-x`  更改为按行输出（默认的顺序为按列输出）

# sort排序

用于文件行排序。`sort 选项 参数`   sort常用选项：

- `-b`  忽略每行前面的空白字符
- `-c`  检查文件是否已经按照顺序排列
- `-u`  忽略相同行
- `-r`  相反的顺序来排序
- `-n`  按照数值的大小排序（例如避免出现10排在2的前面）
- `-d`  忽略英文字母、数字和空格之外的字符
- `-f`  将小写字母视为大写字母排序
- `-i`  忽略040至176之间的ASCII字符外的字符
- `-m`  将几个排序号的文件进行合并
- `-o <输出文件>`  将排序后的结果存入制定的文件
- `-t <分隔字符>`  指定排序时所用的栏位分隔字符（常与-k一起使用）
- `-k <排序的域> `  以指定域的指定区间为标准进行排序
- `+<起始栏位>-<结束栏位>`  由起始栏位到结束栏位的前一栏位排序

##　sort的分隔符和域

分隔符分隔的各个区块即是域。未使用分隔，默认整行就是第1域。

可以使用类似`1.2`的方式表示指定第一个域的第2个字符。未指定域，默认按第一个域的第一个字符排序。

文件friut

> banana:30:5.5
> apple:10:2.5
> pear:90:2.3
> orange:20:3.4

每行以冒号分隔的三项分别表示：名称，数量，价格，对其进行排序处理：

```shell
sort friut  #默认按第一个域的第一个字符排序，依次是apple banana orange pear
sort -k 1.2 friut   #对第一个域的第二个字符排序，依次是banana pear apple orange
sort -n -k 2 -t : friut  #以:为分隔符，对第2域内容（水果价格）以数值进行排序 
```

# uniq去重

用于报告或忽略文件中**相邻的重复行** `uniq 选项 参数`  uniq常用选项：

- `-c`或`--count`  在每列旁边显示该行重复出现的次数
- `-d`或`--repeated`  仅显示重复出现的行列
- `-f<栏位>`或`--skip-fields=<栏位>`  忽略比较指定的栏位
- `-s<字符位置>`或`--skip-chars=<字符位置>`  忽略比较指定的字符
- `-u`或`--unique`  仅显示出一次的行列
- `-w<字符位置>`或`--check-chars=<字符位置>`  指定要比较的字符

注意：uniq只会对**相邻的重复行**进行操作，因此全文去重应配合sort。

```shell
sort file | uniq  #对file内容行排序，再忽略相邻重复行
sort -u file  #同上一行
uniq file  #注意uniq只会删除相邻行中的重复
sort file | uniq -c  #排序去重并统计各行出现的次数
sort file | uniq -d  #将file中的重复行列出
```

# cut列提取

注意：这里的列指的是纵向（垂直方向）

`cut 选项 参数`

常用选项：

- `-f`  指定提取（列）范围（fileds）（类似sort中的-k）
- `-d`  分隔符（**只能是1个字符长度，默认为制表符**），将内容以分隔符分隔成数列，和 `-f` 一起使用  （类似sort中的-t）
- `-c`  以字符(characters)为单位提取
- `-b`  以字节(bytes)为单位提取
- `-n`与`-b`选项连用，不分割多字节字符

指定提取列范围的表示方法（n、x和y均表示一个正整数）

- `n`  第n列的内容
- `x-y`  第x列到第y列间的内容
- `n,x,y`  第n列、第x列和第y列的内容
- `-n`  第1列到第n列的内容
- `n-`  第n列到最后一列的内容

```shell
cut -c-10 /etc/fstab  #查看/etc/fstab文件中每行的第1-10个字符
cut -d ':' -f 1 /etc/passwd  #以:为分隔符 选取分隔出的第1列
cut -d ':' -f 1,3 /etc/passwd #查看所有用户及其id
```

# paste列合并

选项

- `-d` 或`--delimiters` 指定间隔字符 
- `-s`或`--serial` 串列进行而非平行处理（将文件格列内容进行横排）

示例：

文件a

> girl
>
> boy

文件b

> 0
>
> 1

```shell
paste -d ':' a b  #将a内容列合并到b 显然如下
```

> girl:0
> boy:1

```shell
paste -s a b
```

> girl	boy	
> 0	1

# tr字符替换

选项

- `-c`或`--complerment`  取代所有不属于第一字符集的字符
- `-d`或`--delete  `删除所有属于第一字符集的字符
- `-s`或`--squeeze-repeats`  把连续重复的字符以单独一个字符表示
- `-t`或`--truncate-set1`  先删除第一字符集较第二字符集多出的字符

```shell
echo "hello bash" | tr 'a-z' 'A-Z'  #转为HELLO BASH
echo 'cool' | tr -s 'o'  #col
echo -e "I\tam" | tr '\t' ' '  #制表符转换成空格 I am
```

# 流编辑器：sed和awk

- [sed](sed.md)
- [awk](awk.md)