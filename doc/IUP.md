- [IUP中文文档\_个人学习用](#iup中文文档_个人学习用)
- [Guide 引导](#guide-引导)
  - [Getting started 开始使用](#getting-started-开始使用)
  - [Building Applications 创建应用](#building-applications-创建应用)
  - [Building The Library](#building-the-library)
- [Tutorial 教程](#tutorial-教程)
  - [1. Introduction 介绍](#1-introduction-介绍)
  - [2.Hello World](#2hello-world)
    - [2.1 Initialization 初始化](#21-initialization-初始化)
      - [2.1.1 Compiling and Linking 编译和链接](#211-compiling-and-linking-编译和链接)
    - [2.2 Creating a Dialog 构建一个对话(dialog)](#22-creating-a-dialog-构建一个对话dialog)
    - [2.3 Adding Interaction 增加交互](#23-adding-interaction-增加交互)
    - [2.4 Adding Layout Elements 增加布局元素](#24-adding-layout-elements-增加布局元素)
    - [2.5 Improving the Layout 提高布局](#25-improving-the-layout-提高布局)
  - [3.Simple Notepad](#3simple-notepad)
  - [4.Simple Paint](#4simple-paint)
  - [5.Advanced Topics](#5advanced-topics)

# IUP中文文档_个人学习用
原文链接：https://iup.sourceforge.net/

对IUP库官方文档的翻译，方便自己使用。

# Guide 引导
## Getting started 开始使用
## Building Applications 创建应用
## Building The Library


# Tutorial 教程
## 1. Introduction 介绍

IUP有3个每一个用户都应该去理解的概念：**元素（Elements）**, **属性（Attributes）**和**回调（Callbacks）**。

**元素（Elements）**

**属性（Attributes）**用于更改或查询元素的属性。每个元素都有一组影响其行为或外观的属性。每个属性对于不同的元素可能有不同的作用，但通常同名的属性作用是相同的。属性名称总是大写的。但是属性值，如“YES”、“NO”、“TOP”，是不区分大小写的，所以“yes”、“no”、“top”以及其他变体都是可以有效的。

**回调（Callbacks）**是在某些用户接口时间发生时通知应用的函数。通常，回调仅在用户与应用元素交互时才会被调用。如果应用register了回调函数，那么在每次发生该事件时回调都会被调用。

目前为止我们所看到的是对IUP工具包和开发过程中开发者应熟悉的概念的简短总结。从现在开始，我们将展示，如何从一个最简单的例子开始，逐步去开发一个集合了不同资源的复杂IUP应用。

## 2.Hello World
### 2.1 Initialization 初始化
以下代码可以展示如何去打开一个IUP环境并展示一个简单的信息。在之后将详细讲解每一行代码。

in C

```C
#include <stdlib.h>
#include <iup.h>

int main(int argc, char **argv)
{
  IupOpen(&argc, &argv);
  
  IupMessage("Hello World 1", "Hello world from IUP.");
  
  IupClose();
  return EXIT_SUCCESS;
}
```
in Lua
```Lua
require("iuplua")

iup.Message("Hello World 1","Hello world from IUP.")

-- to be able to run this script inside another context
if (iup.MainLoopLevel()==0) then
  iup.MainLoop()
  iup.Close()
end
```

![Hello World 1](https://iup.sourceforge.net/en/tutorial/example2_1.png)

在第一行，我们看到#include了C标准库，这几乎是所有C程序都需要的。接下来是#include 主IUP库，这正是我们第一个示例所需要的。下一行是标准的主函数声明。在运行任何IUP函数之前，必须调用**IupOpen**函数来初始化工具包。下一行使用**IupMessage**函数创建并显示给用户的消息。这个函数接收两个参数：标题(title)和消息(message)。标题(title)将显示在消息窗口的顶部，而消息(message)本身是一个文本消息，将显示给用户。接着，我们有一个**IupClose**函数调用。在运行最后一个IUP函数之后，必须运行**IupClose**，以便工具包可以释放内存并关闭界面系统。最后，程序返回并成功退出。

在Lua中，我们不是**includes**包，而是需要**require**我们使用的包。require调用取代了C中的**IupOpen**函数调用。因此，就像位于代码顶部的include部分一样，IupOpen变成了**require"iuplua"**。这将告知Lua，接下来的代码将使用iuplua包。从C到Lua的另一个重要变化是调用IUP函数的方式。Lua在iuplua包中查找函数时，Lua中的IUP调用使用前缀"iup."而不是"Iup"。所以，在这个例子中，我们有:iup.Message代替**IupMessage**，iup.Close代替**IupClose**。使用Lua与IUP的另一个重要方面是Lua是一种解释型语言，Lua程序从第一行执行到最后一行。因此，不需要创建主函数。我们只需在包含包之后调用iup.Message。然而，在Lua版本的示例中，你会发现有一个调用**iup.MainLoopLevel**。它会检查**iup.MainLoop是否已经被调用**，以避免再次调用它。这只在你的脚本可能从内部的某一个位置开始执行时才有用，对于常规应用程序来说，不需要调用它。

#### 2.1.1 Compiling and Linking 编译和链接
编译和链接一个使用IUP的程序（就像任何其他没有安装在系统上的第三方库一样）需要你指定包含文件和库文件的安装位置。你还需要与iup库进行链接。为了在我们的第一个示例中用单一命令行完成这一操作，命令行指令如下所示：

```
gcc -o example2_1 example2_1.o -liupimglib -liup -L../../../lib/$TEC_UNAME
```

### 2.2 Creating a Dialog 构建一个对话(dialog)
接下来我们稍微修改一下第一个例子，从而添加我们的对话框。

in C
```C
#include <stdlib.h>
#include <iup.h>

int main(int argc, char **argv)
{
  Ihandle *dlg, *label;

  IupOpen(&argc, &argv);

  label =  IupLabel("Hello world from IUP.");
  dlg = IupDialog(IupVbox(label, NULL));
  IupSetAttribute(dlg, "TITLE", "Hello World 2");

  IupShowXY(dlg, IUP_CENTER, IUP_CENTER);

  IupMainLoop();

  IupClose();
  return EXIT_SUCCESS;
}
```
in Lua
```Lua
require("iuplua")

label = iup.label{title = "Hello world from IUP."}
dlg = iup.dialog{
  iup.vbox{label},
  title = "Hello World 2",
}

dlg:showxy(iup.CENTER,iup.CENTER)

-- to be able to run this script inside another context
if (iup.MainLoopLevel()==0) then
  iup.MainLoop()
  iup.Close()
end
```

![Hello World 2](https://iup.sourceforge.net/en/tutorial/example2_2.png)

注意到，我们添加了一行新代码，其中我们声明了Ihandles*类型的变量用于IUP元素。我们还创建了两个不同的变量。一个叫做**dlg**，用于我们的主对话框，另一个叫做**label**，它将包含一个带有问候语的标签。接下来，新的一行创建了一个iup标签控件，并将其与之前声明的label Ihandle关联起来。它唯一的参数是将要在标签内显示的文本。然后是创建我们主对话框的代码行，几乎和创建按钮的方式一样。不同之处在于传递给**IupDialog**函数的参数。它接收另一个函数，该函数将创建一个名为**IupVbox**的组合控件。**IupVbox**是一个将所有传递给它的控件垂直对齐的控件。在这个例子中，我们只传递了一个控件（我们的标签label）和一个NULL，表示我们完成了元素列表。下一行展示了IUP如何改变每个控件的属性。通过调用函数**IupSetAttribute**，程序员将告知哪个控件需要改变属性，哪个属性需要改变，以及该属性将采取的新值。在我们的示例中，我们将主对话框的标题更改为“Hello from IUP Tutorial”。下一个函数叫做**IupShowXY**，它告诉IUP我们需要将主对话框在屏幕的水平和垂直方向上居中显示。接下来是我们程序中调用的最重要的函数之一：**IupMainLoop**。这个函数告诉iup等待事件。否则，程序会继续执行，结束并终止，而不会处理任何事件。继续，注释掉这一行，重新编译你的代码并执行你的程序，你会看到主对话框在屏幕上闪烁，然后程序立即结束。这将是一个有价值的练习。

从最简单的Hello world程序到最复杂的IUP应用，都会包含这相同的代码结构。




### 2.3 Adding Interaction 增加交互
在前面的部分，我们看到了如何去创建一个IUP应用，但是没有任何与对话框的交互。在这一部分，我们将增加一个按钮(button)以增加交互。

in C
```C
#include <stdlib.h>
#include <iup.h>

int btn_exit_cb( Ihandle *self )
{
  IupMessage("Hello World Message", "Hello world from IUP.");
  /* Exits the main loop */
  return IUP_CLOSE;
}

int main(int argc, char **argv)
{
  Ihandle *dlg, *button, *vbox;

  IupOpen(&argc, &argv);
  
  button = IupButton("OK", NULL);
  vbox = IupVbox(
    button,
    NULL);
  dlg = IupDialog(vbox);
  IupSetAttribute(dlg, "TITLE", "Hello World 3");

  /* Registers callbacks */
  IupSetCallback(button, "ACTION", (Icallback) btn_exit_cb);

  IupShowXY(dlg, IUP_CENTER, IUP_CENTER);

  IupMainLoop();

  IupClose();
  return EXIT_SUCCESS;
}
```
in Lua
```Lua
require("iuplua")

function btn_exit_cb(self)
  iup.Message("Hello World Message","Hello world from IUP.")
  -- Exits the main loop
  return iup.CLOSE  
end

button = iup.button{title = "OK", action = btn_exit_cb}

vbox = iup.vbox{button}
dlg = iup.dialog{
  vbox,
  title = "Hello World 3"
}

dlg:showxy(iup.CENTER,iup.CENTER)

-- to be able to run this script inside another context
if (iup.MainLoopLevel()==0) then
  iup.MainLoop()
  iup.Close()
end
```

![hello world 3](https://iup.sourceforge.net/en/tutorial/example2_3.png)

在常规的includes之后，我们看到了几行新代码。这些代码包含了一个名为btn_exit_cb的函数，该函数将被注册为我们的按钮回调函数。除了显示我们在第一个示例中看到的问候消息之外，这个函数没有特别的功能。之后还关闭应用程序，返回代码IUP_CLOSE。

我们添加了一个新的句柄，以清晰的方式处理我们的vbox。接下来是按钮声明。第一个参数是标签的标题，第二个参数是回调的全局名称，现在已不推荐使用，因此我们简单地设置为NULL。接下来的几行是我们的vbox，现在它使用了一个变量。该变量作为参数传递给**IupDialog**函数。

如前所述，回调是由程序员定义的特殊函数，当需要处理事件时，由IUP调用。要创建回调，程序员必须声明一个函数，并在其体内放入他/她希望应用程序在事件发生时执行的任何操作。之后，需要告知IUP这个新函数实际上是一个回调。这是通过调用函数IupSetCallback来完成的。这个调用将告知IUP我们的普通函数btn_exit_cb实际上是一个回调，需要在我们按下按钮时执行。第一个参数是我们的按钮Ihandle，其次是回调的名称和要调用的函数名称，将其转换为Icallback。可用回调的名称可以在每个控件的文档中找到。像属性名称一样，它们总是用大写字母书写。

执行时，应用程序的对话框将显示出来，当用户按下按钮时，它会显示一个问候消息并关闭应用程序。看起来不是什么大事，但通过这个小代码样本，我们已经涵盖了创建IUP应用程序、声明元素和回调以及处理事件的过程。从现在开始，我们将看到更多的IUP控件以及如何使用不同类型的控件来改进我们的应用程序。

在Lua中，我们设置按钮动作回调就像设置一个属性一样。因此，没有调用IupSetCallback。


### 2.4 Adding Layout Elements 增加布局元素

### 2.5 Improving the Layout 提高布局

in C
```C
#include <stdlib.h>
#include <iup.h>

int btn_exit_cb( Ihandle *self )
{
  /* Exits the main loop */
  return IUP_CLOSE;
}

int main(int argc, char **argv)
{
  Ihandle *dlg, *button, *label, *vbox;

  IupOpen(&argc, &argv);
  
  label =  IupLabel("Hello world from IUP.");
  button = IupButton("OK", NULL);
  vbox = IupVbox(
    label,
    button,
    NULL);
  IupSetAttribute(vbox, "ALIGNMENT", "ACENTER");
  IupSetAttribute(vbox, "GAP", "10");
  IupSetAttribute(vbox, "MARGIN", "10x10");
  
  dlg = IupDialog(vbox);
  IupSetAttribute(dlg, "TITLE", "Hello World 5");

  /* Registers callbacks */
  IupSetCallback(button, "ACTION", (Icallback) btn_exit_cb);

  IupShowXY(dlg, IUP_CENTER, IUP_CENTER);

  IupMainLoop();

  IupClose();
  return EXIT_SUCCESS;
}
```
in Lua
```Lua
require("iuplua")

label = iup.label{title = "Hello world from IUP."}
button = iup.button{title = "OK"}

function button:action()
  -- Exits the main loop
  return iup.CLOSE  
end

vbox = iup.vbox{
  label,
  button,
  alignment = "acenter",
  gap = "10",
  margin = "10x10"
}
dlg = iup.dialog{
  vbox,
  title = "Hello World 5"
}

dlg:showxy(iup.CENTER,iup.CENTER)

-- to be able to run this script inside another context
if (iup.MainLoopLevel()==0) then
  iup.MainLoop()
  iup.Close()
end
```
![hello world 5](https://iup.sourceforge.net/en/tutorial/example2_5.png)


![hello world 5a](https://iup.sourceforge.net/en/tutorial/example2_5a.png)
## 3.Simple Notepad
## 4.Simple Paint
## 5.Advanced Topics