关于如何提交博客
===

本README中代码的运行环境为：ubuntu 14.04 Trusty tahr

### 搭建基础环境

首先在本地先搭建相关的环境(可以在虚拟机)，需要git和ruby1.9.3。

```
sudo apt-get install g++
sudo apt-get install git
sudo apt-get install ruby1.9.3
```

然后`sudo gem install bundler`

### 编写文章前的工作

fork本项目，然后clone到本地并切换到source分支（博客都是在source分支上写），以我（andycoder7）为例的话，（fork就不讲了，鼠标点一下就行了）就是：

```
git clone https://github.com/andycoder7/sklcc.github.com.git
cd sklcc.github.com/
git fetch -p
git checkout -t origin/source  
bundle install  
```

### 新建一篇博客

1. 新建一篇博客`rake new_post["YOUR BLOG TITLE"] (标题可以是中文)`，生成的博客会在`source/_posts/YYYY-MM-DD-YOUR-BLOG-TITLE.markdown`文件中  
2. 修改此文件，在categories后添加本篇博客的标签(由于标签云暂不支持中文，所以标签请用英文)，目前的标签有：linux、life   
3. 然后在下一行添加：`author: YOUR NAME`  
4. 然后在`---`的下一行开始编写博客正文内容   
5. 完成后可以使用`rake preview`进行预览  

### 编写完成后

编写完成后提交pull request，其中base是sklcc的source分支，head fork是你的source分支，在本例中head fork是andycoder7的source。然后sklcc会对pull request进行审核和处理，如果没有问题的话会接受request，然后会使用`rake generate`生成jeklly需要的静态文件，然后使用'rake deploy'把文件部署到master分支。此时，你就可以在[sklcc.github.com](http://sklcc.github.com)上看到你的博客啦。
