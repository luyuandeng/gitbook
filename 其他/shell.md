shell脚本
	#!/bin/bash
	echo "Hello World !"
	chmod +x ./test.sh  #使脚本具有执行权限
	./test.sh  #执行脚本


变量
	自定义变量名只能包含数字、字母和下划线
	
	定义：aaa=1
	使用：$aaa
	重新定义：aaa=2
	readonly aaa：只读变量
	unset aaa:删除变量
	类型：局部变量，环境变量，shell变量
		局部：局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量
		环境变量：所有的程序，包括shell启动的程序，都能访问环境变量
		shell变量：shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量
	特殊变量
		$0	当前脚本的文件名
		$n	传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。
		$#	传递给脚本或函数的参数个数。
		$*	传递给脚本或函数的所有参数。
		$@	传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同
		$?	上个命令的退出状态，或函数的返回值。执行成功会返回 0，失败返回 1
		$$	当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。

局部变量与全局变量
	默认
		可改变全局变量值
		外部可访问该变量
	local
		不能改变全局变量值
		外部不能访问该变量

替换
	a=10
	echo -e "Value of a is $a \n"
	变量替换
		$a表示替换成变量a的值 
		${var}	变量本来的值
		${var:-word}	如果变量 var 为空或已被删除(unset)，那么返回 word，但不改变 var 的值。
		${var:=word}	如果变量 var 为空或已被删除(unset)，那么返回 word，并将 var 的值设置为 word。
		${var:?message}	如果变量 var 为空或已被删除(unset)，那么将消息 message 送到标准错误输出，可以用来检测变量 var 是否可以被正常赋值。若此替换出现在Shell脚本中，那么脚本将停止运行。
		${var:+word}	如果变量 var 被定义，那么返回 word，但不改变 var 的值。
	转义字符替换
		-e 表示对转义字符进行替换。如果不使用 -e 选项，将会原样输出
		\\	反斜杠
		\a	警报，响铃
		\b	退格（删除键）
		\f	换页(FF)，将当前位置移到下页开头
		\n	换行
		\r	回车
		\t	水平制表符（tab键） 
		\v	垂直制表符
	命令替换
		aaa=`command`
		将命令结果保存在变量中



运算
	算数运算符、关系运算符、布尔运算符、字符串运算符和文件测试运算符
	expr 是一款表达式计算工具，使用它能完成表达式的求值操作
	算数运算
		val=`expr 2 + 2` 要有空格
		+	加法	`expr $a + $b` 结果为 30。
		-	减法	`expr $a - $b` 结果为 10。
		*	乘法	`expr $a \* $b` 结果为  200。
		/	除法	`expr $b / $a` 结果为 2。
		%	取余	`expr $b % $a` 结果为 0。
		=	赋值	a=$b 将把变量 b 的值赋给 a。
		==	相等。用于比较两个数字，相同则返回 true。	[ $a == $b ] 返回 false。
		!=	不相等。用于比较两个数字，不相同则返回 true。	[ $a != $b ] 返回 true。
	关系运算
		-eq	检测两个数是否相等，相等返回 true。	[ $a -eq $b ] 返回 true。
		-ne	检测两个数是否相等，不相等返回 true。	[ $a -ne $b ] 返回 true。
		-gt	检测左边的数是否大于右边的，如果是，则返回 true。	[ $a -gt $b ] 返回 false。
		-lt	检测左边的数是否小于右边的，如果是，则返回 true。	[ $a -lt $b ] 返回 true。
		-ge	检测左边的数是否大等于右边的，如果是，则返回 true。	[ $a -ge $b ] 返回 false。
		-le	检测左边的数是否小于等于右边的，如果是，则返回 true。	[ $a -le $b ] 返回 true。
	布尔运算
		!	非运算，表达式为 true 则返回 false，否则返回 true。	[ ! false ] 返回 true。
		-o	或运算，有一个表达式为 true 则返回 true。	[ $a -lt 20 -o $b -gt 100 ] 返回 true。
		-a	与运算，两个表达式都为 true 才返回 true。	[ $a -lt 20 -a $b -gt 100 ] 返回 false。
	字符串运算
		=	检测两个字符串是否相等，相等返回 true。	[ $a = $b ] 返回 false。
		!=	检测两个字符串是否相等，不相等返回 true。	[ $a != $b ] 返回 true。
		-z	检测字符串长度是否为0，为0返回 true。	[ -z $a ] 返回 false。
		-n	检测字符串长度是否为0，不为0返回 true。	[ -n $a ] 返回 true。
		str	检测字符串是否为空，不为空返回 true。	[ $a ] 返回 true。
	文件测试运算符
		-b file	检测文件是否是块设备文件，如果是，则返回 true。	[ -b $file ] 返回 false。
		-c file	检测文件是否是字符设备文件，如果是，则返回 true。	[ -b $file ] 返回 false。
		-d file	检测文件是否是目录，如果是，则返回 true。	[ -d $file ] 返回 false。
		-f file	检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。	[ -f $file ] 返回 true。
		-g file	检测文件是否设置了 SGID 位，如果是，则返回 true。	[ -g $file ] 返回 false。
		-k file	检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。	[ -k $file ] 返回 false。
		-p file	检测文件是否是具名管道，如果是，则返回 true。	[ -p $file ] 返回 false。
		-u file	检测文件是否设置了 SUID 位，如果是，则返回 true。	[ -u $file ] 返回 false。
		-r file	检测文件是否可读，如果是，则返回 true。	[ -r $file ] 返回 true。
		-w file	检测文件是否可写，如果是，则返回 true。	[ -w $file ] 返回 true。
		-x file	检测文件是否可执行，如果是，则返回 true。	[ -x $file ] 返回 true。
		-s file	检测文件是否为空（文件大小是否大于0），不为空返回 true。	[ -s $file ] 返回 true。
		-e file	检测文件（包括目录）是否存在，如果是，则返回 true。	[ -e $file ] 返回 true。


字符串
	单引号	
		单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
		单引号字串中不能出现单引号（对单引号使用转义符后也不行）。
	双引号
		双引号里可以有变量
		双引号里可以出现转义字符
	拼接字符串
		your_name="qinjx"
		greeting="hello, "$your_name" !"
		greeting_1="hello, ${your_name} !"
		echo $greeting $greeting_1
	字符串长度
		${#string}
	提取子字符串
		${string:1:4}

数组
	array_name=(value0 value1 value2 value3)
	array_name[0]=value0 array_name[1]=value1
	${array_name[2]} 第三个元素
	${array_name[*]} 数组所有元素
	${#array_name[*]} 数组长度

echo
	echo "\"It is a test\"" 双引号也可以省略
	echo "$name It is a test"
	echo "${mouth}-1-2009" 变量与其它字符相连的话，需要使用大括号
	echo "It is a test" > myfile 显示结果重定向至文件
	echo "It is a test" >> myfile 显示结果追加到文件
	echo `date` 显示命令执行结果


printf
	格式化输出， 是echo命令的增强版
	printf "%d %s\n" 1 "abc" 
	printf %s abcdef
	printf "%s %s %s\n" a b c d e f g h i j
	%s 默认NULL代替，%d 用 0 代替


if...else
	if ... fi 语句；
	if ... else ... fi 语句；
	if ... elif ... else ... fi 语句。
	if [ expression ]
	then
	   Statement(s) to be executed if expression is true
	fi

	if [ expression 1 ]
	then
	   Statement(s) to be executed if expression 1 is true
	elif [ expression 2 ]
	then
		Statement(s) to be executed if expression 1 is true
	else
	fi
	if test $[2*3] -eq $[1+5]; then echo 'The two numbers are equal!'; fi;
	expression 和方括号[ ]，if和[]之间必须有空格
	test 命令用于检查某个条件是否成立，与方括号([ ])类似


循环
	for
		for loop in 1 2 3 4 5
		do
		    echo "The value is: $loop"
		done
		for str in 'This is a string'
		for file in $HOME/.bash*
	while
		COUNTER=0
		while [ $COUNTER -lt 5 ]
		do
		    COUNTER='expr $COUNTER + 1'
		    echo $COUNTER
		done
		while循环可用于读取键盘信息 while read FILM 按<Ctrl-D>结束循环
	until
		与while相反
	break：跳出所有或指定循环 
	continue：跳出当前循环

输入输出
	默认从标准输入设备(stdin)获取输入，将结果输出到标准输出设备(stdout)显示
	命令的输出不仅可以是显示器，还可以很容易的转移向到文件，这被称为输出重定向。
	一般情况下，每个 Unix/Linux 命令运行时都会打开三个文件：
		标准输入文件(stdin)：stdin的文件描述符为0，Unix程序默认从stdin读取数据。
		标准输出文件(stdout)：stdout 的文件描述符为1，Unix程序默认向stdout输出数据。
		标准错误文件(stderr)：stderr的文件描述符为2，Unix程序会向stderr流中写入错误信息。
		默认情况下，command > file 将 stdout 重定向到 file，command < file 将stdin 重定向到 file。
	command 2 > file   stderr 重定向到 file
	command > file 2>&1  将 stdout 和 stderr 合并后重定向到 file
	command < file1 >file2    command 命令将 stdin 重定向到 file1，将 stdout 重定向到 file2
	command > /dev/null 如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到 /dev/null
	wc -l 命令计算 document 的行数
		$wc -l << EOF
		    This is a simple lookup program
		    for good (and bad) restaurants
		    in Cape Town.
		EOF


shell函数
	function 函数名() {
	    语句
	    [return]
	}
	语句部分可以是任意的Shell命令，也可以调用其他的函数
	break中断函数的执行
	函数名直接调用，不用括号
	函数名 参数1 参数2 参数3 参数4，当函数执行时，$1 对应 参数1，其他依次类推。
	载入另一个文件的函数 source test.sh
	默认情况下，变量具有全局作用域，如果想把它设置为局部作用域，可以在其前加入local，局部变量，使得函数在执行完毕后，自动释放变量所占用的内存空间




文件内容处理
	for line in `cat data.dat`
	do 
	    echo "File:${line}"
	done
	cat data.dat | awk 'for(i=2;i<NF;i++) {printf $i}}'
	cat data.dat | while read line
	do
	    echo "File:${line}"
	done


管道
	

awk
wc
sed
cut
let
(())


linux 命令
http://www.runoob.com/linux/linux-command-manual.html