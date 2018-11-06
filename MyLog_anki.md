### 1 目录结构 & 测试代码

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

### 2 调试，大致了解对象层次

vscode中创建main.py，启动调试。Debug console输出

```
➜  anki git:(master) ✗ cd /home/qian/project/learn/public/anki ; env "PYTHONIOENCODING=UTF-8" "PYTHONUNBUFFERED=1" python /home/qian/.vscode/extensions/ms-python.python-2018.9.2/pythonFiles/experimental/ptvsd_launcher.py 38212 /home/qian/project/learn/public/anki/tests/main.py
```

```
def Collection(path, lock=True, server=False, sync=True, log=False):
	db = DB(path)   # path是后缀名为anki2的文件路径
```

storage.py中的Collection函数

1. 连接数据库db
2. 返回collection.py中的_Collection对象。`col = _Collection(db, server, log)`

_Collection构造函数中

```
        self.media = MediaManager(self, server)
        self.models = ModelManager(self)
        self.decks = DeckManager(self)
        self.tags = TagManager(self)
```

其中的XXManager来自xx.py

DB类的_db适配了sqlite3.Connections

deck是_Collection

综上，collection.py的_Collection是入点。

收获：对象的层层管理

### 3 查看note的过程

deck生成note，使用不同flag预览note的不同card。

```
deck = getEmptyCol()
f = deck.newNote()
# all templates
cards = deck.previewCards(f, 2)
```

在collection.py的Outline中输入Note查找

collection的addNote、rmNotes实际上是新增、删除跟note相关的card。

```python
def newNote(self, forDeck=True):
        "Return a new note with the current model."
        return anki.notes.Note(self,self.models.current(forDeck))
```



全局搜索newNode。

除了在tests函数中，另外在addcards.py、models.py中调用，两个都在aqt中。2018-11-06 17:08:38

