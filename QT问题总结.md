>编号：
>
>内容：QT开发问题总结 
>
>创建时间：2024-09-30
>
>更新时间：2024-11-01
>
>标签：
>
>备注：Qt5.12.0

---



# Qt5.12.0

### connect连接时，槽函数拥有多个重载，需要指明其中一个

![image-20241012171938552](C:\Users\Jun\AppData\Roaming\Typora\typora-user-images\image-20241012171938552.png)

解决：

添加静态类型转换，匹配目标重载函数。

```c++
static_cast<void (sideBarMenu::*)(QStackedWidget *)>
			//返回类型	 函数名，*全选即可    参数表
```

```c++
connect(toolButton, &QToolButton::clicked, this, static_cast<void (sideBarMenu::*)(QStackedWidget *)>(&sideBarMenu::menuSelected));
```



### qt_metacall不是xx的成员错误

通常是由于多重继承时继承顺序不正确导致的。当你的类继承自 `QObject` 并且使用了 `Q_OBJECT` 宏时，`QObject` 必须是第一个基类。这是因为 Qt 的元对象系统（包括信号和槽机制）依赖于 `QObject` 的元数据，而这些元数据是在编译时通过 MOC（Meta-Object Compiler）生成的。

即 QObject写在前



### 无法定位到某个具体文件的语法错误：缺少">"

![image-20241101114306623](C:\Users\Jun\AppData\Roaming\Typora\typora-user-images\image-20241101114306623.png)

![image-20241101114334119](C:\Users\Jun\AppData\Roaming\Typora\typora-user-images\image-20241101114334119.png)

最后出现Signal and slot的报错，发现是某个connect中信号和槽的参数对应不上。



### 换行符失效

1. HTML 模式下的换行

如果你的 `QTextBrowser` 控件处于 HTML 模式（通常是文本内容使用了HTML的标签），换行符 `\n` 可能不会被解释为换行。在 HTML 中，换行通常通过 `<br>` 标签来实现。如果你的 `QTextBrowser` 设置为解析 HTML，你应该使用 `<br>` 而不是 `\n`：

```cpp
textBrowser->append("第一行<br>第二行");
```

或者，如果你使用纯文本模式，确保 `QTextBrowser` 的 `setOpenExternalLinks` 和 `setHtml` 方法正确设置：

```cpp
textBrowser->setHtml("第一行<br>第二行");
textBrowser->setOpenExternalLinks(false);
```

2. 纯文本模式下的换行

如果你的 `QTextBrowser` 控件处于纯文本模式，确保你使用的是正确的换行符。在大多数平台上，`\n` 是正确的换行符。如果你使用的是 Windows 系统，可能需要使用 `\r\n`：

```cpp
textBrowser->append("第一行\n第二行");
```

3. 编码问题

如果你从文件或其他来源读取文本并添加到 `QTextBrowser`，确保文本的编码正确。`QTextBrowser` 应该能够正确处理 UTF-8 编码的文本，但如果文本编码不正确，可能会导致显示问题：

```cpp
QString text = ...; // 确保这是 UTF-8 编码的文本
textBrowser->append(text);
```

> 注意通常都是由于HTML模式下失效



### widget->Layout()->addWidget()报访问位置冲突错误

错误原因大多数是由于widget还不存在layout，需要通过setLayout的方式来添加。或者在UI界面手动先设置layout



### 信号和槽之间传递自定义类型失败，并且没有报错

原因：信号和槽缺少处理该自定义类型的能力。

解决：注册自定义类型

进行注册

```c++
#include <QMetaType.h>
Q_DECLARE_METATYPE(testInfo);				//声明为metatype自定义类型
qRegisterMetaType<YourStruct>("YourStruct");//注册自定义类型，必须在使用到的信号和槽的连接之前声明，声明在cpp文件中
```

通常注册后即可，也可以使用Qvariant接收

发射信号时使用QVariant

```c++
YourStruct yourStruct;
QVariant var;
var.setValue(yourStruct);
emit yourSignal(var); // 发射 QVariant 类型的信号
```

> 某些内置类型如QList都有可能处理不了，需要注册。



### QSqlDatabase: QMYSQL driver not loaded问题

旧版Qt会存在动态库缺少的问题

https://blog.csdn.net/qq_37529913/article/details/109850670

`MySql`需要`libmysql.dll` `qsqlmysql.dll`等动态库，并且版本需要对应安装的数据库版本。前者在安装的mysql lib目录中，后者需要对qt的文件进行配置和编译获取。



### Qt 设置为窗口模式下，出现控制台模式下没有的报错 class QAxFactory * __cdecl qax_instantiate(void)无法解析

原因：未知

解决：QT模块只勾选`ActivateQt container`，不勾选`ActiveQt Server`

![image-20241202162526899](C:\Users\Jun\AppData\Roaming\Typora\typora-user-images\image-20241202162526899.png)
