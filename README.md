## Use Todo.txt to manage your life job

1. [Todo.txt][1]
支持多种编程语言的开发

2. [Todo.txt add on][3]
cygwin制作linux环境，需要把bash文件dos2unix.exe转换文件，否则出现故障。

    + [t graph][5] 
    + [t mit][6]
       日度 月度  季度 年度计划的指定工具
    + [t mitf 1d][7]
       带时间段的定义task
    + [t top   t view  t edit][8]
       view比较好用
    + [t xp][9]
       更好显示已完成任务
    + [t birdseye][10]
        产量报告
        A Python script birdseye.py (called with the birdseye action) analyzes the todo.txt and done.txt files to generate a report of completed and incomplete items in every context and project. (Requires Python and both birdseye.py and birdseye files to run.)
    + [t sync][11]
同步，需要[t commit][12]
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
