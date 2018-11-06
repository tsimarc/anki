### 1

anki目录下没有gui成分。aqt下含有gui成分。

`/home/qian/project/learn/public/anki/tools/tests.sh`可以指定测试tests下某个文件（里面都是test开头的函数，没有`__main__`)

```bash
(venv) ➜  anki git:(master) ✗ ls tools
anki-wait.bat  build_ui.sh  runanki.system.in  tests.sh
(venv) ➜  anki git:(master) ✗ tools/tests.sh 
Call with coverage=1 to run coverage tests
tools/tests.sh: line 31: nosetests: command not found
```

the hard way中的execise中提到[nosetests - zofia_enjoy的博客 - CSDN博客](https://blog.csdn.net/zofia_enjoy/article/details/72594185)

nose官网已经废弃 [Note to Users — nose 1.3.7 documentation](https://nose.readthedocs.io/en/latest/)

> Nose has been in maintenance mode for the past several years and will likely cease without a new person/team to take over maintainership. New projects should consider using [Nose2](https://github.com/nose-devs/nose2), [py.test](http://pytest.org/), or just plain unittest/unittest2.

```bash
pip install nose

(venv) ➜  anki git:(master) ✗ which nosetests
/home/qian/project/learn/public/anki/venv/bin/nosetests
```

印象：没有引号、双引号都能展开$，甚至有x$x这种类似拼接字符串的效果。

启发

```
BIN="$(cd "`dirname "$0"`"; pwd)"
export PYTHONPATH=${BIN}/..:${PYTHONPATH}
```

[Shell 教程 | 菜鸟教程](http://www.runoob.com/linux/linux-shell.html)

`export PYTHONPATH=.`这样sys.path第二个总是当前目录的绝对路径，没用。

export PYTHONPATH=`pwd`

`git remote add -h`



