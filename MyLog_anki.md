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

### 3 单步调试GUI启动过程

解压apkg文件后

```bash
➜  anki tree
.
├── jouyou.apkg
└── jouyou.apkg_FILES
    ├── 0
    ├── collection.anki2
    └── media
```

导入时，注意到涉及~/.local文件夹。

调试runanki。

在aqt/init中

```
# profile manager
    from aqt.profiles import ProfileManager
    pm = ProfileManager(opts.base)
```

opts.base默认为空字符。ProfileManager区分不同平台，Linux下为

```
dataDir = os.environ.get(
                "XDG_DATA_HOME", os.path.expanduser("~/.local/share"))
```

然后实例化`class AnkiApp(QApplication):`

检查一些设置后，创建aqt/main.py中的主窗口`mw = aqt.main.AnkiQt(app, pm, opts, args)`

执行到`app.exec`启动的窗口的初始界面上，已经含有deck。应该是在上面窗口的创建过程中已经读取了。

init函数中主要执行两个函数

1. setupUI，setupUI中调用各种setup方法。
2. setupProfile。调用链：loadProfile 》 loadCollection 》_loadCollection

```
    def _loadCollection(self):
        cpath = self.pm.collectionPath()

        self.col = Collection(cpath, log=True)
```

### 4

2018-11-07 10:43:58 sqlitebrowser中查看jouyou.anki2。注意到有Paragrma。表、表字段都要schema。

数据集中card、note两个表。note中的csum都是很大的数字。card中的did不知何意，看到的值都一样。

sqlitebrowser用Qt5写成，有表格、图表。2018-11-07 10:50:44

### 5

anki web上点`Basic phone mode`调到`Simple Study Mode`

尝试观察html、网络请求。

web版切换卡片后，右上角的数字可能不会立即更新。对数字颜色、含义不解。

"Show Answer" > reiviewer.py > 

嵌入了html和事件调用，追溯到web里的reviewer.js里的pycmd('ans')

webviwe.py中`AnkiWebPage(QWebEnginePage):`中

```python
script = QWebEngineScript()
        script.setSourceCode(js + '''
            var pycmd;
```

调试，窗口没出现前，setupUI》setupMainWindow，完成了绑定

窗口出现口，sync进度条跳过后，出现主窗口，空白。

此时cmd是domDone

```
_onBridgeCmd (/home/qian/project/learn/public/anki/aqt/webview.py:334)
_onCmd (/home/qian/project/learn/public/anki/aqt/webview.py:85)
cmd (/home/qian/project/learn/public/anki/aqt/webview.py:27)
_run (/home/qian/project/learn/public/anki/aqt/__init__.py:347)
run (/home/qian/project/learn/public/anki/aqt/__init__.py:262)
<module> (/home/qian/project/learn/public/anki/runanki:4)
```

看见首页前，出现了好像4词domDone

点deck名，cmd是'open:1541609815897'，接着又出现domDone

进入deck页。点”Study Now"，cmd是study

两次domDone后，显示卡片首页。（期间窗口空白）

“Show Answer”，cmd是“ans”，按钮点击状态下阻塞

看不出来怎么跳过去的

```
_linkHandler (/home/qian/project/learn/public/anki/aqt/reviewer.py:286)
_onBridgeCmd (/home/qian/project/learn/public/anki/aqt/webview.py:346)
_onCmd (/home/qian/project/learn/public/anki/aqt/webview.py:85)
cmd (/home/qian/project/learn/public/anki/aqt/webview.py:27)
_run (/home/qian/project/learn/public/anki/aqt/__init__.py:347)
run (/home/qian/project/learn/public/anki/aqt/__init__.py:262)
<module> (/home/qian/project/learn/public/anki/runanki:4)
```

继续进入，生成js，最终在webviews中eval生成的js更新页面。

点了‘Again'按钮，ease1

```
def _linkHandler(self, url):
        if url == "ans":
            self._getTypedAnswer()
        elif url.startswith("ease"):
            self._answerCard(int(url[4:]))
        elif url == "edit":
            self.mw.onEditCurrent()
        elif url == "more":
            self.showContextMenu()
        else:
            print("unrecognized anki link:", url)
```



