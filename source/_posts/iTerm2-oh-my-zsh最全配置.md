---
title: iTerm2+oh-my-zsh最全配置
date: 2021-09-30 10:57:47
tags: [iTem2, oh-my-zsh, Mac]
---
# 安装前先看效果图

![效果图](https://img-blog.csdnimg.cn/48edcbdf2daa430b887244c624919247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbnhpbW8=,size_16,color_FFFFFF,t_70#pic_center)
<!--more-->

## 一.安装[iTem2](https://iterm2.com/)
点击标题即可下载,如果没有安装brew,执行下面命令一键安装brew并自动替换为国内镜像源
```bash
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/4e2f117e99af46c8b79e76597b379062.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbnhpbW8=,size_16,color_FFFFFF,t_70#pic_center)

## 二.安装[oh-my-zsh](https://ohmyz.sh/)
可能会失败,多执行几遍到成功为止
```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)" 
```

## 三.安装[Powerline](https://powerline.readthedocs.io/en/latest/installation.html)
首先需要安装pip命令(也可自行安装Python自带pip),再安装Powerline
```bash
sudo easy_install pip
pip install powerline-status
```
或者
```bash
brew install python3
pip3 install powerline-status
```

## 四.安装 [Meslo](https://github.com/powerline/fonts) 字体库
可直接复制下面代码片执行
```bash
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts
```
安装完毕后,进入到iTem2配置页面进行配置:
设置背景颜色
![背景](https://img-blog.csdnimg.cn/c2c9e4f7c9ef467090534689aea6006d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbnhpbW8=,size_16,color_FFFFFF,t_70#pic_center)
设置字体样式
![字体样式](https://img-blog.csdnimg.cn/08449346236945b9b6351609ae45a77d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbnhpbW8=,size_16,color_FFFFFF,t_70#pic_center)
设置command+.全局呼出
![全局呼出](https://img-blog.csdnimg.cn/80e0d88ab25c432c88075c3d2937bd9f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbnhpbW8=,size_16,color_FFFFFF,t_70#pic_center)
## 五.安装agnoster主题
无需安装,直接配置即可

```bash
cd ~
vim ~/.zshrc
```
找到ZSH_THEME,参数改成"agnoster"
![修改参数](https://img-blog.csdnimg.cn/ddd26a64d4c947f4be88438178e2489c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbnhpbW8=,size_16,color_FFFFFF,t_70#pic_center)
## 六.安装语法高亮

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
在根目录`.zshrc`插入(注意{your_system_name}需要替换你的系统用户名):

```bash
source /Users/{your_system_name}/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
plugins=(
  git
  zsh-syntax-highlighting
)
```
## 七.安装代码补全插件
1、`zsh-completions`
```bash
git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:=~/.oh-my-zsh/custom}/plugins/zsh-completions
```
在根目录`.zshrc`插入:

```bash
plugins=(
  git
  zsh-completions
)
autoload -U compinit && compinit
```
2、`zsh-autosuggestions`：补全的是历史输入的命令，点击方向键->即可补全

```bash
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```
在根目录`.zshrc`插入:

```bash
plugins=(
  git
  zsh-autosuggestions
)
```
因为之前调整过背景色,这边默认是灰色,实际效果展示不出,所以要修改配置文件,调整自动补全的底色:
打开文件:
```bash
vim ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
```
修改内容:

```bash
ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=blue'
```
![修改配色](https://img-blog.csdnimg.cn/884c4f8542a4404aae1115adda067684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbnhpbW8=,size_16,color_FFFFFF,t_70#pic_center)
**注意:安装上述所有内容后,一定要执行`source ~/.zshrc`使配置生效**
## 八.常用命令
```bash
# 查看shell
cat /etc/shells
# 更改shell
chsh -s /bin/zsh
# 查看当前shell
echo $SHELL
```

 [1]参考文章地址:https://www.jianshu.com/p/246b844f4449
