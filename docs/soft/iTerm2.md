## 文本复制

在iTerm2 中，选中即复制，所以在iTerm2的session中不用再去 ⌘+c ,可直接将选中的文本复制到剪切板中去，通常选中有以下两种方法：

1. 使用鼠标选择。

2. 使用 ⌘+f 搜索，查找内容会高亮显示，通过tab / shift+tab 扩大选中范围，快捷键可在Profiles > Keys 中设置。

 

## 智能选中

双击选中，三击选中整行，四击智能选中

按住⌘键

1. 可以拖拽选中的字符串；
2. 点击 url：调用默认浏览器访问该网址；
3. 点击文件：调用默认程序打开文件；
4. 如果文件名是filename:42，且默认文本编辑器是 Mac vim将会直接打开到这一行；
5. 点击文件夹：在 finder 中打开该文件夹；
6. 同时按住opt键，可以以矩形选中。

 

## Tab 窗口面板管理

Mac下默认的终端窗口分屏不是很好使，当初就是因为这个原因，才使用iTerm2,那么接下来看下iTerm2窗口面板分割功能。

Tab纵向分割：⌘+d



Tab横向分割：⌘+shift+d



切换Tab中的pane：⌘ + [  或者 ⌘+ opt + arrow

关闭panel：⌘ + w

最大化Tab中的pane，隐藏本Tab中的其他pane：⌘+ shift +enter , 再次还原



新建Tab ：⌘ + t

Tab 切换：⌘ + arrow 或者 ⌘+shift + [

改变Tab的顺序：⌘ + shift + arrow

快速切换到Tab上：⌘ + Num

最大化Tab ： ⌘ + enter  再次还原

窗口太多，可以使用 ⌘ + / 快速定位到光标所在位置



一屏显示所有窗口：⌘ + alt+ e



## 标记跳转

类似编辑器的mark工具，iTerm2也可以在命令行位置设置标记

设置标记：⌘ + shift + m

跳转到上个标记：⌘ + shift + j

多个标记切换：⌘ + shift + arrow

 

## 及时回放

某个交互命令会覆写屏幕上的输入，之前的历史信息可能会被覆盖掉，无法查看，iterm2 这个及时回放功能，会记录历史输入，输出，有点类似视频录制。



进入回放：⌘ + opt + b 

方向键控制时间 ：arrow  

退出回放：esc

 

## 其他

自动填充：⌘ + ； 命令补全提示 



 

查找：⌘ + f

打开粘贴历史：⌘ + shift + h  



打开最近目录： ⌘ + alt + /

显示鼠标引导： ⌘ + alt + ；  鼠标所在行高亮显示

设置Terminal热键：pref > keys


设置触发操作，比如输入关键字，将背景颜色高亮