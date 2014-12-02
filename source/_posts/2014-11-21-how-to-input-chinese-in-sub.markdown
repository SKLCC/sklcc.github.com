---
layout: post
title: "如何在Sublime Text 3中输入中文"
date: 2014-11-21 23:53:48 +0800
comments: true
categories: sublime linux
author: andy
---

From Andy at SuZhou sklcc.com


### 版本说明

 - Ubuntu 14.04.1 Trusty Tahr 64bit
 - Sublime Text 3065
 - fcitx 4.2.8.4
 - 搜狗输入法 1.1.0.0037
 - gcc 4.8.2

### 流程

#### 1. 安装sublime text 3

进入sublime text 3官方[下载页面](http://www.sublimetext.com/3)，选择[Ubuntu 64 bit](http://c758482.r82.cf2.rackcdn.com/sublime-text_build-3065_amd64.deb)，下载完成后双击安装即可。

<!--more-->

{% img /images/how_to_input_chinese_in_subl/after_install_sub3.png %} 

#### 2. 安装依赖软件

由于我们接下来需要编译一个共享库，所以需要用到‘build-essential’和‘libgtk2.0-dev’这两个软件。

`sudo apt-get install build-essential libgtk2.0-dev`

#### 3. 编写sublime-imfix.c文件

该文件自己创建，文件位置随意，文件内容如下：

```
/*
 * sublime-imfix.c
 * Use LD_PRELOAD to interpose some function to fix sublime input method support for linux.
 * By Cjacker Huang <jianzhong.huang at i-soft.com.cn> *
 *
 * gcc -shared -o libsublime-imfix.so sublime_imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC
 * LD_PRELOAD=./libsublime-imfix.so sublime_text
 */
#include <gtk/gtk.h>
#include <gdk/gdkx.h>

typedef GdkSegment GdkRegionBox;

struct _GdkRegion
{
    long size;
    long numRects;
    GdkRegionBox *rects;
    GdkRegionBox extents;
};

GtkIMContext *local_context;

void
gdk_region_get_clipbox (const GdkRegion *region,
                        GdkRectangle    *rectangle)
{
    g_return_if_fail (region != NULL);
    g_return_if_fail (rectangle != NULL);

    rectangle->x = region->extents.x1;
    rectangle->y = region->extents.y1;
    rectangle->width = region->extents.x2 - region->extents.x1;
    rectangle->height = region->extents.y2 - region->extents.y1;
    GdkRectangle rect;
    rect.x = rectangle->x;
    rect.y = rectangle->y;
    rect.width = 0;
    rect.height = rectangle->height;

    //The caret width is 2;
    //Maybe sometimes we will make a mistake, but for most of the time, it should be the caret.
    if (rectangle->width == 2 && GTK_IS_IM_CONTEXT(local_context)) {
        gtk_im_context_set_cursor_location(local_context, rectangle);
    }
}

//this is needed, for example, if you input something in file dialog and return back the edit area
//context will lost, so here we set it again.

static GdkFilterReturn event_filter (GdkXEvent *xevent, GdkEvent *event, gpointer im_context)
{
    XEvent *xev = (XEvent *)xevent;

    if (xev->type == KeyRelease && GTK_IS_IM_CONTEXT(im_context)) {
        GdkWindow *win = g_object_get_data(G_OBJECT(im_context), "window");

        if (GDK_IS_WINDOW(win)) {
            gtk_im_context_set_client_window(im_context, win);
        }
    }

    return GDK_FILTER_CONTINUE;
}

void gtk_im_context_set_client_window (GtkIMContext *context,
                                       GdkWindow    *window)
{
    GtkIMContextClass *klass;
    g_return_if_fail (GTK_IS_IM_CONTEXT (context));
    klass = GTK_IM_CONTEXT_GET_CLASS (context);

    if (klass->set_client_window) {
        klass->set_client_window (context, window);
    }

    if (!GDK_IS_WINDOW (window)) {
        return;
    }

    g_object_set_data(G_OBJECT(context), "window", window);
    int width = gdk_window_get_width(window);
    int height = gdk_window_get_height(window);

    if (width != 0 && height != 0) {
        gtk_im_context_focus_in(context);
        local_context = context;
    }

    gdk_window_add_filter (window, event_filter, context);
}

```

进入该文件所在的目录，然后编译该文件，输入命令如下：

```
gcc -shared -o libsublime-imfix.so sublime-imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC
```

最后我们将会在当前目录下得到libsublime-imfix.so这个共享库文件。

#### 4. 安装搜狗输入法

Ubuntu14.04 默认安装的是ibus，目前在sublime text3中只能支持fcitx的中文输入，所以我们需要安装搜狗输入法。**注意：ibus不能卸载，否则容易出现系统崩溃.** fcitx安装方法也很简单，首先进入[搜狗官网](http://pinyin.sogou.com/linux/)，如果是ubuntu14.04（64bit）的直接下载[deb包](http://pinyin.sogou.com/linux/download.php?f=linux&bit=64)，然后双击安装即可。

然后我们需要进入system setting（系统设置） -> Language Support（语言支持），把键盘输入改为fcitx

{% img /images/how_to_input_chinese_in_subl/set_language.png %}

此时默认输入法已经切换为fcitx了，fcitx中是没有搜狗输入法的，所以我们需要进入*Fcitx Configuration*添加。在终端中输入：`fcitx-configtool`，然后点击左下角的添加，去除Only Show Current Language（只显示当前语言）的选项，然后在搜索框中输入*so*，如下图所示，然后点击左下角的“+”即可：

{% img /images/how_to_input_chinese_in_subl/add_sougou.png %}

然后我们需要去除系统的切换输入法的快捷键，否则会与fcitx的快捷键冲突，具体方法是进入system setting（系统设置） -> Keyboard（键盘） -> Shortcuts（快捷方式） -> Typing（打字）页面，然后把Switch to next source改为Disable（用鼠标点击那一项，然后按backspace）

{% img /images/how_to_input_chinese_in_subl/remove_hotkey.png %}

**全部完成后需要logout重新登陆一下配置才会生效，当然reboot也是可以的。**

#### 5. 中文输入

此时我们应该已经可以正常的切换输入法来输入中英文了。然后我们在终端中输入命令：

`LD_PRELOAD=./libsublime-imfix.so subl             #subl是安装好SublimeText 3后的程序启动命令`

如果一切正常，在启动之后，搜狗输入法就可以正常输入了。

#### 6. 为了方便

由于每次都要输入LD_PRELOAD太不方便了，所以我们可以直接编辑/usr/bin/subl文件

首先把libsublime-imfix.so拷贝到～/.config/sublime-text-3/目录下，然后`sudo vi /usr/bin/subl`，在exec前面加上LD_PRELOAD，例如在我的/usr/bin/subl文件中的内容是：

```
#!/bin/sh
LD_PRELOAD=/home/andy/.config/sublime-text-3/libsublime-imfix.so exec  /opt/sublime_text/sublime_text "$@"
```

如果不习惯用命令的小伙伴，也可以修改subl的快捷方式图标，具体方法参见*参考文件1*

#### 7. BUGs

在sublime text中输入中文时，选词框会在页面左下脚，而不是跟随光标。

#### 8. Tips

在sublime text 3 中安装Package Control的方法：使用```ctrl+` ```然后把下面代码贴入输入框中：

```
import urllib.request,os,hashlib; h = '7183a2d3e96f11eeadd761d777e62404' + 'e330c659d4bb41d3bdf022e94cab3cd0'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by) 
```

如果顺利的话，此时就可以在Preferences菜单下看到Package Settings和Package Control两个菜单了。 

### 参考文献

1. [在Ubuntu 14.04中使SublimeText 3支持中文输入法](http://blog.csdn.net/cywosp/article/details/32350899)
2. [Package Control Installation](https://sublime.wbond.net/installation#st3)

