> Linux的命令，用suffix= "--help"就知道用法了

# multi-process unzip
```bash
ls *.zip | xargs -P $(nproc) -I{} 7z x {} -y
```

# real-time fresh
```bash
watch -n 2 -d "free -h"
```
* top 查看那CPU
* 系统管理小能手

<img width="2292" height="249" alt="image" src="https://github.com/user-attachments/assets/947eb0c0-a9a3-4879-b4ff-c4468f5de6a1" />

# 恢复默认.bashrc

```bash
cp /etc/skel/.bashrc ~/.bashrc
```

# snap 关闭自动更新

```bash
sudo snap refresh --hold
```

# 把文件file转为todos

```
printf "%s\n" *>> read.md
```

# 终端编辑器(vi,当nano不可用)

* 删除整行——dd/D
* 跳转到最后一行——G
* 跳转到第一行——gg
* 跳转到下一行——k
* 跳转到上一行——o

# 软件包(snap+apt)

## 查看软件版本

```sh
sudo snap info okular
```

# 环境变量查看+修改

## $PATH

```sh
export PATH=$PATH:/directory
```

# regular strings

[正则表达式 &#8211; 语法 | 菜鸟教程](https://www.runoob.com/regexp/regexp-syntax.html)

## grammar

正则表达式的基本语法包括多种元素和构造，用于匹配字符串中的特定模式。以下是一些常见的基本语法元素：

### 1. 字符类

- **`.`**：匹配任意单个字符（除了换行符）。
- **`[abc]`**：匹配字符集中的任意一个字符（例如，`[abc]` 匹配 `a`、`b` 或 `c`）。
- **`[^abc]`**：匹配不在字符集中的任意一个字符（例如，`[^abc]` 匹配除了 `a`、`b` 和 `c` 之外的字符）。
- **`[a-z]`**：匹配范围内的任意字符（例如，`[a-z]` 匹配小写字母）。

### 2. 量词

- **`*`**：匹配前面的元素零次或多次。
- **`+`**：匹配前面的元素一次或多次。
- **`?`**：匹配前面的元素零次或一次。
- **`{n}`**：匹配前面的元素恰好 n 次。
- **`{n,}`**：匹配前面的元素至少 n 次。
- **`{n,m}`**：匹配前面的元素至少 n 次，但不超过 m 次。

### 3. 边界匹配

- **`^`**：匹配字符串的开始。
- **`$`**：匹配字符串的结束。
- **`\b`**：匹配单词边界（例如，空格、标点符号等）。
- **`\B`**：匹配非单词边界。

### 4. 转义字符

- **`\`**：用于转义特殊字符，使其失去特殊含义。例如，`\.` 匹配字面上的点字符。

### 5. 分组和捕获

- **`(abc)`**：匹配 `abc` 并将其作为一个组捕获。
- **`(?:abc)`**：匹配 `abc` 但不捕获（非捕获组）。
- **`(?P<name>abc)`**：匹配 `abc` 并将其捕获为命名组 `name`。

### 6. 逻辑或

- **`|`**：表示逻辑或。例如，`a|b` 匹配 `a` 或 `b`。

### 7. 特殊字符

- **`\d`**：匹配任何数字，等价于 `[0-9]`。
- **`\D`**：匹配任何非数字字符。
- **`\w`**：匹配任何字母数字字符（包括下划线），等价于 `[a-zA-Z0-9_]`。
- **`\W`**：匹配任何非字母数字字符。
- **`\s`**：匹配任何空白字符（空格、制表符、换行符等）。
- **`\S`**：匹配任何非空白字符。

### `_字符需要转义吗?`

在正则表达式中，`_`（下划线）是一个普通字符，不需要转义。它可以直接用作匹配字符。例如，正则表达式 `\w` 会匹配字母、数字和下划线，因此 `_` 是被包含在内的。

### 总结

正则表达式的基本语法提供了强大的文本匹配能力。理解这些基本构造可以帮助你编写有效的正则表达式来处理各种文本处理任务。

##  os与shell

| 层级    | 举例              |
| ----- | --------------- |
| 操作系统  | Unix / Linux    |
| Shell | bash / zsh / sh |
| 命令    | ls / cp / rm    |
| 程序    | gcc / python    |

### bash下的通配符

| 通配符  | 含义                |
| ---- | ----------------- |
| `*`  | 匹配任意长度字符串（不含 `/`） |
| `?`  | 匹配任意 **单个字符**     |
| `[]` | 匹配字符集合，如 `[a-z]`  |
| `{}` | 花括号展开，如 `{a,b}`   |
| `~`  | 用户家目录             |
| `**` | 递归匹配（bash 4+，需开启） |



## example

1. 在当前目录下匹配'depth_env'开头，'_cam2_frame1.jpg'结尾，中间是数字
   
   * 注意find查找的结果，会以'./'开头
     
     ```bash
     find . -type f -regex   '\./depth_env[0-9]+_cam2_frame1.jpg'
     ```

2. cam-0001-\d\d\d\d.png

```bash
cp $(ls | grep 'cam-0001-[0-9]{4}.png')  rgb/
```

* find /ls 不支持{n}匹配，也不支持\d

# 命令行输出作为另一命令的输入

## 1. 命令替换(Best!)

* 可以使用反引号（`）或 $() 语法来将 find 命令的输出作为 cp 命令的参数。

```bash
cp $(find . -name "*.txt") /path/to/destination/
```

## 2. --exec

find 命令的 -exec 选项允许你对找到的每个文件执行一个命令。你可以在 -exec 后面直接使用 cp 命令。

```bash
find . -name "*.txt" -exec cp {} /path/to/destination/ 
```

## 3. 管道和xargs

* 可以使用 xargs 命令将 find 的输出传递给 cp 命令。xargs 会将输入转换为命令行参数。

```bash
find . -name ".*.txt" | xargs -I {} cp {} /path/to/destination/
```

## example

* 查找后复制

```
cp $(find . -type f -regex   '\./depth_env[0-9]+_cam2_frame1.jpg')  test/
```

# 查找

* 按正则表达式

```bash
find . -iregex "*.txt"
```

* 按文件名

```bash
find . -name "*txt"
```

* 按文件+路径名

```bash
find -wholename "/path/to/file"
```

* 具体见

```bash
find --help
```

# wget下载

```bash
wget https://example.com/file.zip          # 下载单个文件
wget -O renamed_file.zip https://example.com/file.zip  # 指定保存文件名
wget -c https://example.com/bigfile.iso   # 断点续传
wget -r --no-parent https://example.com/path/  # 递归下载（不跨越父目录）
```

# 查看目录大小

```bash
du -a -h -d 1
```

* -a= output hidden file

* -h=humankind, -d=recursive depth

# ibus

* 故障重启

```bash
ibus   restart
ibus-daemon   -drx
```


# 挂载-升级权限

```bash
sudo umount /media/sky/SSD

# 选项A：使用默认权限（推荐）
sudo mount -o rw,uid=1000,gid=1000,iocharset=utf8 /dev/sda1 /media/sky/SSD
# 选项B：给予所有用户完全权限（最宽松）
sudo mount -o rw,uid=1000,gid=1000,umask=000,iocharset=utf8 /dev/sda1 /media/sky/SSD
```

# awk

## Bash 中的 AWK 用法

AWK 是一种强大的文本处理工具，适合用于数据的格式化、分析和报告。它特别适合处理结构化文本，如 CSV 文件和日志文件。在 Bash 中，使用 AWK 可以简化复杂的文本处理任务。

### 基本语法

AWK 的通用命令格式如下：

```bash
awk 'pattern { action }' filename
```

- **pattern**：匹配输入数据的模式，省略时对所有行操作。
- **action**：在匹配到模式的行上执行的动作，如果省略，默认是打印整行。

### 举例说明

#### 打印整行

```bash
awk '{print}' file.txt
```

#### 打印特定列

```bash
awk '{print $1, $2}' file.txt
```

#### 使用分隔符

AWK 默认使用空格作为字段分隔符，可以使用 `-F` 指定其他分隔符：

```bash
awk -F',' '{print $1, $2}' file.txt
```

#### 打印行数

```bash
awk '{print NR, $0}' file.txt
```

此命令将打印行号和整行内容。

#### 计算列的总和

```bash
awk '{sum += $1} END {print sum}' file.txt
```

#### 格式化输出

```bash
awk '{printf "%-10s %-10s\n", $1, $2}' file.txt
```

### 条件与循环

AWK 允许使用条件判断来选择性地输出特定行。

#### 过滤特定条件

打印第一列大于2的行：

```bash
awk '$1 > 2' file.txt
```

### 使用内置变量

AWK 提供一些内置变量，例如：

- **`NR`**：当前行号。
- **`NF`**：当前行的字段数。
- **`$0`**：完整的当前行。

示例：

```bash
awk '{print NR, NF, $0}' file.txt
```

### 使用外部脚本

对于较长的 AWK 程序，您可以将代码写入一个文件，并使用 `-f` 选项加载：

```bash
awk -f script.awk file.txt
```

### 小结

AWK 是一个极为灵活的工具，适用于多种文本处理场景。无论是简单的字段提取，还是复杂的数据分析，AWK 都能够高效处理。希望这些示例能帮助你在 Bash 中有效地使用 AWK！如果你有具体的应用场景或者问题，欢迎继续询问！

# xargs

```bash
(legged-rsl) chenlei@118:~/Project/DeepSDF$ echo 'one two three' | xargs -n 1 -d " " -I {} echo "Item: {}"
Item: one
Item: two
Item: three
```
* -d " "按空格划分;
* -I 按“行”处理, 不再按空格拆分
* -n 每次只传x个参数
