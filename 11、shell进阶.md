# 11、shell进阶

## 1 数组

### 1.1数组基础

```
cpu_load.sh
service_manager.sh
```

单行定义

```shell
root@rocky9:shell # num_list=(123 456 789)
root@rocky9:shell # echo ${num_list}
123
root@rocky9:shell # echo ${num_list[0]}
123
root@rocky9:shell # echo ${num_list[1]}
456
root@rocky9:shell # echo ${num_list[2]}
789
```

多行定义

```shell
root@rocky9:shell # class_one=(
> zhansan
> lisi
> wangwu
> sunliu
> zhaoqi
> )
root@rocky9:shell # echo ${class_one}
zhansan
root@rocky9:shell # echo ${class_one[0]}
zhansan
root@rocky9:shell # echo ${class_one[1]}
lisi
root@rocky9:shell # echo ${class_one[2]}
wangwu
root@rocky9:shell # echo ${class_one[3]}
sunliu
```

单元素定义

```shell
root@rocky9:shell # mix_list[0]=nihao
root@rocky9:shell # mix_list[2]=123
root@rocky9:shell # mix_list[4]=hello
root@rocky9:shell # echo ${mix_list}
nihao
root@rocky9:shell # echo ${mix_list[1]}

root@rocky9:shell # echo ${mix_list[2]}
123
root@rocky9:shell # echo ${mix_list[4]}
hello
root@rocky9:shell # echo ${mix_list[2]}
123
root@rocky9:shell # echo ${mix_list[3]}

```

批量定义多元素

```shell
root@rocky9:shell # string_list=([0]="value-1" [3]="value-2")
root@rocky9:shell # echo ${string_list[@]}
value-1 value-2
root@rocky9:shell # echo ${string_list[0]}
value-1
root@rocky9:shell # echo ${string_list[1]}

root@rocky9:shell # echo ${string_list[3]}
value-2
```

命令定义

```shell
root@rocky9:shell # touch {1..5}.sh
root@rocky9:shell # ls
1.sh  2.sh  3.sh  4.sh  5.sh
root@rocky9:shell # file_array=$(ls *.sh)
root@rocky9:shell # echo ${file_array} 
1.sh 2.sh 3.sh 4.sh 5.sh
root@rocky9:shell # echo ${file_array[@]} 
1.sh 2.sh 3.sh 4.sh 5.sh
root@rocky9:shell # echo ${file_array[1]} 

root@rocky9:shell # echo ${file_array[0]} 
1.sh 2.sh 3.sh 4.sh 5.sh
注意：
 对于命令的数组创建来说，它只有一个元素
```

### 1.2 数组取数

```
读取数组元素值可以根据元素的下标值来获取，语法格式如下：
	${array_name[index]}
	${array_name[@]:起始位置:获取数量}
注意：
 	获取具体的元素内容，指定其下标值，从0开始
 	获取所有的元素内容，下标位置写"@"或者"*"
 
在找内容的时候，有时候不知道数组的索引都有哪些，我们可以基于如下方式来获取，数组的所有索引：
   ${!array_name[index]}
注意：
 	获取所有的元素位置，下标位置写"@"或者"*"
 
获取数组长度的方法与获取字符串长度的方法相同，格式如下：
 	${#array_name[index]}
注意：
 	获取具体的元素长度，指定其下标值，从0开始
 	获取所有的元素个数，下标位置写"@"或者"*"
 	
从系统中获取所有的数组
 	declare -a
```

基于索引查找内容

```
root@rocky9:shell # num_list=(123 456 789)
root@rocky9:shell # echo ${num_list}
123
root@rocky9:shell # echo ${num_list[@]}
123 456 789
root@rocky9:shell # echo ${num_list[*]}
123 456 789
root@rocky9:shell # echo ${num_list[-1]}
789
root@rocky9:shell # echo ${num_list[-2]}
456
root@rocky9:shell # echo ${num_list[@]:1:1}
456
root@rocky9:shell # echo ${num_list[@]:1:2}
456 789
root@rocky9:shell # echo ${num_list[@]:0:3}
123 456 789
```

基于内容获取元素(获取下标)

```shell
root@rocky9:shell # echo ${!num_list[@]}
0 1 2
root@rocky9:shell # echo ${!mix_list[@]}
0 2 4
```

获取数组长度

```
root@rocky9:shell # echo ${#mix_list[@]}
3
root@rocky9:shell # echo ${#num_list[@]}
3
root@rocky9:shell # echo ${#file_array[@]}
1
root@rocky9:shell # echo ${#num_list[2]}
3
root@rocky9:shell # echo ${#num_list[1]}
3
```

获取系统所有数组

```
root@rocky9:shell # declare -a
declare -a BASH_ARGC=([0]="0")
declare -a BASH_ARGV=()
declare -a BASH_COMPLETION_VERSINFO=([0]="2" [1]="11")
declare -a BASH_LINENO=()
declare -a BASH_REMATCH=([0]="\${!mix" [1]="\${!" [2]="{!" [3]="mix")
declare -a BASH_SOURCE=()
declare -ar BASH_VERSINFO=([0]="5" [1]="1" [2]="8" [3]="1" [4]="release" [5]="x86_64-redhat-linux-gnu")
declare -a DIRSTACK=()
declare -a FUNCNAME
declare -a GROUPS=()
declare -a PIPESTATUS=([0]="1")
declare -a class_one=([0]="zhansan" [1]="lisi" [2]="wangwu" [3]="sunliu" [4]="zhaoqi")
declare -a mix_list=([0]="nihao" [2]="123" [4]="hello")
declare -a num_list=([0]="123" [1]="456" [2]="789")
declare -a string_list=([0]="value-1" [3]="value-2")
```

### 1.3数组变动

```
元素内容替换：
 	array_name[index]=值
注意：
 	在修改元素的时候，index的值一定要保持准确
元素部分内容替换，可以参考字符串替换格式：
 	${array_name[index]/原内容/新内容}
注意：
 默认是演示效果，原数组未被修改，如果真要更改需要结合单元素内容替换
```

```shell
root@rocky9:shell # echo ${num_list[@]} 
123 456 789
root@rocky9:shell # num_list[2]=aaa
root@rocky9:shell # echo ${num_list[@]} 
123 456 aaa
root@rocky9:shell # echo ${num_list[2]/aa/lualu-}
lualu-a
root@rocky9:shell # echo ${num_list[@]} 
123 456 aaa
root@rocky9:shell # num_list[2]=${num_list[2]/aa/lualu-}
root@rocky9:shell # echo ${num_list[@]} 
123 456 lualu-a
```

元素删除

```
删除单元素
 	unset array_name[index]
删除整个数组
 	unset array_name
```

```shell
oot@rocky9:shell # num_list[3]=147
root@rocky9:shell # num_list[4]=258
root@rocky9:shell # num_list[5]=369
root@rocky9:shell # echo ${num_list[@]} 
123 456 lualu-a 147 258 369
root@rocky9:shell # unset num_list[2]
root@rocky9:shell # echo ${num_list[@]} 
123 456 147 258 369
root@rocky9:shell # unset num_list[2]
root@rocky9:shell # echo ${num_list[@]} 
123 456 147 258 369
root@rocky9:shell # unset num_list[1]
root@rocky9:shell # echo ${num_list[@]} 
123 147 258 369
root@rocky9:shell # echo ${!num_list[@]} 
0 3 4 5
```

```shell
root@rocky9:shell # unset num_list 
root@rocky9:shell # echo ${!num_list[@]} 

root@rocky9:shell # echo ${num_list[@]} 

```

### 1.1.4属组关联

```
定制索引数组 - 数组的索引是普通的数字
declare -a array_name
 	- 普通数组可以不事先声明,直接使用
 
定制关联数组 - 数组的索引是自定义的字母
 	declare -A array_name
 	- 关联数组必须先声明,再使用
```

定制索引数组

```shell
root@rocky9:shell # declare -a course
root@rocky9:shell # declare -a | grep course
declare -a course

root@rocky9:shell # course=(yuwen shuxue yingyu)
root@rocky9:shell # declare -a | grep course
declare -a course=([0]="yuwen" [1]="shuxue" [2]="yingyu")
```

定制关联属组

```shell
root@rocky9:shell # declare -A sourse
root@rocky9:shell # sourse=([yuwen]="99" [shuxue]="98" [yingyu]="97")
root@rocky9:shell # declare -A | grep sourse
declare -A sourse=([yuwen]="99" [shuxue]="98" [yingyu]="97" )
root@rocky9:shell # echo ${sourse[@]}
99 98 97
root@rocky9:shell # echo ${!sourse[@]}
yuwen shuxue yingyu
root@rocky9:shell # echo ${sourse[yuwen]}
99
root@rocky9:shell # echo ${sourse[shuxue]}
98
root@rocky9:shell # echo ${sourse[yingyu]}
97
```

## 2 分支逻辑

shell逻辑

```
条件逻辑 - 多分支执行命令块
 	- if控制语句
    - case控制语句
    - select控制语句
循环逻辑 - 多循环执行命令块
 	- for控制语句
 	- while控制语句
 	- until控制语句
逻辑控制 - 命令块执行过程中，精细化控制
 	- continue控制
 	- break控制
 	- exit控制
 	- shift控制
```

### 2.1 if

```
single_branch_if.sh
double_branch_if.sh
multi_branch_if.sh
service_manager_if.sh
service_manager_if.sh
simple_jumpserver_if.sh
```



```
单路决策 - 单分支if语句
 样式：
        if [ 条件 ]
        then
            指令
        fi
    特点：
   	单一条件，只有一个输出
```

```
双路决策 - 双分支if语句
 样式：
        if [ 条件 ]
        then
            指令1
        else
            指令2
        fi
    特点：
   单一条件，两个输出
```

```
多路决策 - 多分支if语句
 	样式：
 		if [ 条件 ]
        then
            指令1
        elif [ 条件2 ]
        then
            指令2
        else
            指令3
        fi
 特点：
   n个条件，n+1个输出
```

```
单行命令写法
 if [ 条件1 ]; then 指令1; elif [ 条件2 ]; then 指令2; ... ; else 指令n; fi
```

```
关键点解读：
 1 if 和 then 配套使用
 2 if 和末尾的 fi 顺序反写
```

```
shell的if语句中关于条件判断这块内嵌了如下几种测试表达式语句：
 	[ 表达式 ] 		针对通用的判断场景
 	[[ 表达式 ]] 		针对扩展的判断场景
 	(( 命令 )) 		(())代替let命令来测试数值表达式
```

### 2.2 casc

```
service_manager_case.sh
kubernetes_manager_case.sh
```

```
case 变量名 in
    值1)
      指令1
    ;;
    ...
    值n)
      指令n
    ;;
esac

注意：
    首行关键字是case，末行关键字esac
    选择项后面都有 )
    每个选择的执行语句结尾都有两个分号;
```

## 3、循环逻辑

```
for_hand_list.sh
for_define_list.sh
for_cmd_list.sh
for_arg_list.sh
for_add_user.sh
for_host_check.sh
```

```
循环逻辑语法解析：
 	关键字 [ 条件 ]
 	do
 	执行语句
 	done
 
注意：
 	这里的关键字主要有四种：
 		for - 循环遍历一个元素列表
 		while - 满足条件情况下一直循环下去
 		until - 不满足条件情况下一直循环下去
```

### 3.1 for

```
场景：遍历列表
    for 值 in 列表
    do
       执行语句
    done
    
注意：
    ”for” 循环总是接收 “in” 语句之后的某种类型的字列表
    执行次数和list列表中常数或字符串的个数相同，当循环的数量足够了，就自动退出
```

```
样式1：手工列表
 - 1 2 3 4 5 6 7
样式2：定制列表
 - {1..7}
样式3：命令生成
 - $(seq 1 7)
样式4：脚本参数
 - $@ $*
```

### 3.2 while

```
场景：只要条件满足，就一直循环下去
    while [ 条件判断 ]
    do
       执行语句
    done
    
注意：
    条件支持的样式 命令、[[ 字符串表达式 ]]、(( 数字表达式 ))
    true是一个特殊的条件，代表条件永远成立
```

### 3.3 until基础

```
场景：只要条件不满足，就一直循环下去
    until [ 条件判断 ]
    do
       执行语句
    done
    
注意：
    条件支持的样式 命令、[[ 字符串表达式 ]]、(( 数字表达式 ))
    false是一个特殊的条件，代表条件永远不成立
```

### 3.4 循环控制

```
continue控制
 	- 满足条件的情况下，临时停止当前的循环，直接进入到下一循环
break控制
 	- 满足条件的情况下，提前退出当前的循环
exit控制
 	- 直接退出当前循环的程序
shift控制
 	- 依次从循环列表中读取读取内容，并将读取的内容从列表中剔除
```

```
exit
	exit在shell中是一个特殊的程序退出信号，不仅仅可以直接退出当前程序，还可以设定退出后的状态
返回值，使用方式如下：
 	exit num
注意：
 	1 在脚本中遇到exit命令，脚本立即终止；终止退出状态取决于exit命令后面的数字
 	2 如果exit后面无数字,终止退出状态取决于exit命令前面命令执行结果
```

```powershell
root@kang:shell # cat exit_multi_for.sh
#!/bin/bash
#功能：exit退出脚本

# 外层循环遍历1-5
for var1 in {1..5}
do
   # 内层循环遍历1-5
   for var2 in {a..d}
   do
      #判断退出条件，var1是2或者var2是c就退出内层循环
      if [ $var1 -eq 2 -o "$var2" == c ]
      then
        exit 111
      else
        echo "$var1 $var2"
      fi
    done
done
```



```
break
	break命令是在处理过程中终止循环的一种简单方法。可以使用break命令退出任何类型的循环，包括for、while、until等。
	
break主要有两种场景的表现样式：
 	单循环场景下，break是终止循环
 		- 仅有一层 while 、for、until等
 	嵌套循环场景下，break是可以终止内层循环和外层循环。
 		- 存在多层while、for、until嵌套等
```

```powershell
break语法格式：
  for 循环列表
  do
    ...
    break num
   done
 注意：
 单循环下，break就代表退出循环
 多循环下，break的num大于嵌套的层数，就代表退出循环
```

```powershell
root@kang:shell # cat break_single_while.sh 
#!/bin/bash
# 功能：break退出单层循环
while true
do
  read -p "输入你的数字，最好在1 ~ 5：" aNum
  case $aNum in
    1|2|3|4|5)
      echo "你的数字是 $aNum!"
    ;;
    *)
      echo "你选择的数字没在1 ~ 5，退出！"
      break
    ;;
  esac
done
```

```powershell
root@kang:shell # cat break_multi_in_while.sh 
#!/bin/bash
# 功能：break退出内层循环

# 外层循环遍历1-5
for var1 in {1..5}
do
  # 内层循环遍历a-d
  for var2 in {a..d}
  do
    # 判断退出条件，var1是2或者var2是c
    if [ $var1 -eq 2 -o "$var2" == "c" ]
    then
      break
    else
      echo "$var1 $var2"
    fi
  done
done
```

continue

```
continue命令是在处理过程中跳出循环的一种简单方法。可以使用continue命令跳出当前的循环直接进入到下一个循环，包括for、while、until等。

continue主要有两种场景的表现样式：
 	单循环场景下，continue是跳出当前循环
 		- 仅有一层 while 、for、until等
 	嵌套循环场景下，continue是可以跳出内层循环和外层循环。
 		- 存在多层while、for、until嵌套等
```

```
continue语法格式：
  for 循环列表
  do
    ...
    continue num
  done
 注意：
 单循环下，continue就代表跳出当前循环
 多循环下，continue的num就代表要继续的循环级别
```

```powershell
root@kang:shell # cat continue_single_while.sh 
#!/bin/bash
# 功能：continue退出单层循环
while true
do
  read -p "输入你的数字，最好在 1~5 ：" aNum
  case $aNum in
    1|2|3|4|5)
      echo "你的数字是 $aNum!"
    ;;
    *)
      echo "你选择的数字没在1~5,退出！"
      continue
    ;;
  esac
done
```







20250316
