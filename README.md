## Use Todo.txt to manage your life job

1. [Todo.txt][1]
支持多种编程语言的开发

2. [Todo.txt add on][3]
cygwin制作linux环境，需要把bash文件dos2unix.exe转换文件，否则出现故障。

    + [t graph][5] 
    + [t mit][6]
       t mit 日期 任务信息：日期包含着具体的日子[today,mon,tue,wed,thr,fri,sat,2017.05.05]、月份[jan,feb]、季度[q1,q2,q3,q4]、年[2017,2018]等信息
       【日度】 【月度】  【季度】 【年度】计划的指定工具
       t mit 可以查看【四度】计划表，相当于计划汇总表
       t mit @f708 显示所有地点在f708的计划表，四度都会显示出来，但是速度慢一些
       t mit not @f708  显示不在f708的计划表[可惜的是，还无法支持地点分层]
       t mit rm和t mit mv类似于 t rm和t mv.
    + [t mitf 1d][7]
       带时间段的定义task
       t mitf 1w "@f708 +multiAxis ..."
       t mitf 1m "@f708 +multiAxis ..."
       t mitf 1d "@f708 +multiAxis ..."
    + [t top   t view  t edit][8]
       top显示头20天信息。

       view比较好用
       一般用t view project 显示【项目分层】的计划表，t view context显示【地点相关分层】的计划表，由于时间相关的
       任务得在每条任务中添加t:2017-05-03的标签，所以t view date,t view today ,t view tomorrow,t view future,t view past等时间相关的任务在任务中没有t:开头的时间标签都认为是无效，得手动进行添加了,This is shortage,希望改进。

       为此改进如下：只要添加t:2017-05-03日期规范即可，于是写了如下【简单】脚本【简单又复杂2h】【注意dos2unix.exe】
``` sh
#count=1;
for quartern in `grep -no "[^ :]*{[^ ]\+}" todo.txt|tr "." "-"|awk -F"[{-}]" '{printf("%s%s\n",$1,$2)}'`;do 
    item=`echo $quartern|cut -d ":" -f1`
    quarter=`echo $quartern|cut -d ":" -f2`
    date1=`echo $quarter|sed 's/\(.*\)00-00/\101-01/'|
    sed 's/\(.*\)[qQ]1-00/\101-01/'|
    sed 's/\(.*\)[qQ]2-00/\104-01/'|
    sed 's/\(.*\)[qQ]3-00/\107-01/'|
    sed 's/\(.*\)[qQ]4-00/\110-01/'|
    awk -F"-" '{if($3==00) printf("%s%s%s%s%s\n","t:",$1,"-",$2,"-01"); else printf("%s%s%s%s%s%s\n","t:",$1,"-",$2,"-",$3)}'`
#    echo ${item}
#    echo $date1
    todo.sh append ${item} $date1
#    echo ${count} 
#    echo ${date1} 
#    count=$((count+1));

done
```
【思路】：
+ 抓取todo.txt的行号和时间信息，目标结合mit的时间，所以是大括号里头的时间，得到对应行号和时间信息
+ 获取行号，可以提供给todo.sh append中的$item选项，即计划好 ，对应todo.txt的行号
+ 获取时间，并进行特殊日期处理，比如年、季度、月份的处理
+ 最后调用todo.sh append即可，刚好在对应任务插入一行字符串，perfect解决

因为重复上述脚本，会不断增加t:日期，于是又写了一个【删除】脚本
``` sh
sed -ie "s/t:.*$//g" todo.txt
```
删除todo.txt目录下的该文件， 从t:开始到行尾都删除掉。增删就都完成了，这样就可以使用t view date, t view tomorrow, t view today , t view past ,t view future,
t view nodate等。
![todo][14]
![result][15]

```
https://raw.githubusercontent.com/jueqingsizhe66尝试访问本地图片 没用！
改为： https://raw.github.com/jueqingsizhe66尝试访问本地图片 有用！
github内部机制,似乎无效
https://github.com/jueqingsizhe66/Todo/blob/master/finalResult2.png
有效了
```

    + [t xp][9]
       更好显示【已完成】任务
       t xp -o 10 
       显示10天内完成的任务
    + [t birdseye][10]
        t birdseye: 生成产量报告,完成量和未完成量的统计
        A Python script birdseye.py (called with the birdseye action) analyzes the todo.txt and done.txt files to generate a report of completed and incomplete items in every context and project. (Requires Python and both birdseye.py and birdseye files to run.)
    + [t sync][11]
同步，需要[t commit][12],算了不用这个了,手动提交。
    + [t due][13]
修改了due.py,配合上mit的时间格式.
``` python
#!/usr/bin/env python
"""
due.py
Python 2 script for todo.txt add-on
Created by Rebecca Morgan 2017-03-10
Copyright (c) 2017 Rebecca Morgan. All rights reserved.
"""

import os
import sys
from datetime import datetime, timedelta
import re

def main(todo_dir, future_days = 0):
    # Prepare lists to store tasks
    overdue = list()
    due_today = list()
    due_future = list()

    # Open todo.txt file
    with open(os.path.join(todo_dir,'todo.txt'), 'r') as f:
        content = f.readlines()
        date = datetime.today()

        # Loop through content and look for due dates, assuming the key due: is used and standard date format
        for i, task in enumerate(content):
            #match = re.search(r'due:\d{4}.\d{2}.\d{2}', task)
            match = re.search(r'\{(\d{4}).(\d{2}).(\d{2})\}', task)
#            if match.group(3) == '00':
#                match.group(3) =='01'
#            if match.group(2) == '00':
#                match.group(2) == '01'
            if match is not None:
                if match.group(3) == '00':
#                    match.group(3) ='01'
                    matchUpdate=match.group(1)+"."+match.group(2)+"."+"01"
               #     print matchUpdate
               #     print 'hello'
                    if match.group(2) == '00':
                        #match.group(2) = '01'
                        matchUpdate=match.group(1)+"."+"01"+"."+"01"
               #         print matchUpdate
                else:
                    matchUpdate=match.group(1)+"."+match.group(2)+"."+match.group(3)

#                print match.group()
#                print match.group(2)
#                print match.group(1)
                #date = datetime.strptime(match.group()[4:], '%Y.%m.%d').date()
                #date = datetime.strptime(match.group()[1:len(match.group())-1], '%Y.%m.%d').date()
                date = datetime.strptime(matchUpdate, '%Y.%m.%d').date()

                # Add matching tasks to list with line number
                if date < datetime.today().date():
                    overdue.append(str(i+1).zfill(2) + " " + task)
                elif date == datetime.today().date():
                    due_today.append(str(i+1).zfill(2) + " " + task)
                elif date < datetime.today().date() + timedelta(days=future_days+1):
                    due_future.append(str(i+1).zfill(2) + " " + task)

    # Print to console
    if len(overdue) > 0:
        print "==============================="
        print "Overdue tasks:"
        print "==============================="
        for task in overdue:
            print task,
    if len(due_today) > 0:
        print "==============================="
        print "Tasks due today:"
        print "==============================="
        for task in due_today:
            print task,
    if len(due_future) > 0:
        print "==============================="
        print "Tasks due in the next " + str(future_days) + " days:"
        print "==============================="
        for task in due_future:
            print task,

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print "Usage: due.py [TODO_DIR] <future_days>"
        sys.exit(1)

    if os.path.isdir(sys.argv[1]):
        if len(sys.argv) is 3:
            main(sys.argv[1], int(sys.argv[2]))
        else:
            main(sys.argv[1])
    else:
        print "Error: %s is not a directory" % sys.argv[1]
        sys.exit(1)

```


3. [Todo.txt vim plugin][4]

``` vim
Sorting tasks:
 <localleader>s  Sort the file
 <localleader>s+  Sort the file on +Projects
 <localleader>s@  Sort the file on @Contexts
 <localleader>sd  Sort the file on dates
 <localleader>sdd  Sort the file on due dates

Edit priority:
 <localleader>j  Decrease the priority of the current line
 <localleader>k  Increase the priority of the current line
 <localleader>a  Add the priority (A) to the current line
 <localleader>b  Add the priority (B) to the current line
 <localleader>c  Add the priority (C) to the current line

Date:
 <localleader>d  Set current task's creation date to the current date
 date<tab>  (Insert mode) Insert the current date

Mark as done:
 <localleader>x  Mark current task as done
 <localleader>X  Mark all tasks as done
 <localleader>D  Move completed tasks to done.txt

```
4. [The video about Todo.txt][2]


5. todo原始命令的使用

**注意事先得对t进行alias，并执行todo的命令补全功能shell,具体官网看说明**

+ t add @f708 +multiAxis finish paper
表示 添加一个t add命令表示添加一个命令  @后面跟着【地点】 +后面跟着【项目名】，之后的都叫做任务信息(task info)

+ t list
或者t ls,表示【列出】todo.txt内容的所有【任务】

+ t listall
<font color="red">把已完成和未完成的任务通通显示出来，并在报告的结尾做一个【简短的报告】，well done</font>.

+ t p 18 B
表示给任务号为18的【添加】一个B的【权限】(权限按照字母表A-Z都可以)，可以通过t list显示所有任务信息，然后通过t lsp显示所有的
带权限的任务。 
相反地，t depri 或者t dp表示【去除】相应任务的【权限】

+ t lsprj
表示【显示】任务中涉及到的【项目】(注意区分显示带权限的任务),也可以写作t listproj,类似的t lsc表示显示项目中所有涉及到地址(即@打头的),也可以写作t listcon
+ t rm  1
```
$ t rm 1
Delete '(A) due:2017.05.02  @home do some job'?  (y/n)
```
显示是否【删除】一个task no 的【任务】。类似于t del 1。

+ t do 1
完成任务1，并把该条语句信息【包含完成时间】提交到done.txt中

+ t report
生成一个report.txt,然后把对应日期里面的【未完成】的计划数、【已完成】的计划数写在后面两位

+ t
单单一个t会显示所有的项目名字和地址名字

+ t listfile
或者t lf列出所有todo.txt目录相关的文件

```
Files in the todo.txt directory:
done.txt
hey.txt
report.txt
todo.txt
```

+ t archive
定期可以做一个【归档】,会把todo.txt done.txt中的空行去除，生成.bak文件

+ t prepend 1 "hello"
或者t prep 1 hello，引号可以省略，表示在一个任务号前面添加信息
```
$ t prep 1 fuck
1 fuck due:2017.05.02  @home do some job
```
相反地，t append 1 "fuck" 或者t app 1 fuck表示在任务1的末尾添加fuck字段。


[1]:https://github.com/ginatrapani/todo.txt-cli 
[2]:https://vimeo.com/3263629 
[3]:https://github.com/ginatrapani/todo.txt-cli/wiki/Todo.sh-Add-on-Directory#mit-most-important-task 
[4]:https://github.com/freitass/todo.txt-vim 
[5]:https://github.com/timpulver/todo.txt-graph 
[6]:https://github.com/codybuell/mit 
[7]:https://github.com/linduxed/todo.txt-mitf 
[8]:https://github.com/Thann/todo-cli-plugins 
[9]:https://github.com/gr0undzer0/xp 
[10]:https://github.com/ginatrapani/todo.txt-cli/tree/addons/.todo.actions.d 
[11]:https://github.com/ginatrapani/todo.txt-cli/wiki/Todo.sh-Add-on-Directory#sync-sync-state-between-local-and-remote-git-repository 
[12]: https://raw.githubusercontent.com/fnd/todo.txt-cli/extensions/commit
[13]:https://github.com/rebeccamorgan/due 
[14]:https://github.com/jueqingsizhe66/Todo/blob/master/finalResult.png
[15]:https://github.com/jueqingsizhe66/Todo/blob/master/finalResult2.png

