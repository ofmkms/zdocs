x270s

# about insert key
	because insert key is function key, so in linux term, <br> 
	use use fn+ctrl insert(End) for copy <br> 
	use fn+shift insert(End) for paste

	ctrl insert  === fn + ctrl + end
	ctrl insert  === fn + ctrl + end

------------

# install chinese input in  ubuntu 16.04.04

## use fcitx（ubuntu16.04中文版默认的是fcitx输入法）
### 增加Chinese语言包
	System Settings--> Language Support--> Install/Remove Languages  add Chinese(simplified)
    	说明在首次打开Language Surpport页面是系统会默认安在fcitx需要的软件包等。
### 选择keyboard input method system是 fcitx
### 系统重启
```
reboot
```
### 配输入法
	输入法-->ConfigureFcitx-->Input Method-->"+"-->增加pinyin、Sunpinyin或者Googlepinyin。
### 测试


## use ibus 
### 增加Chinese语言包
	System Settings--> Language Support--> Install/Remove Languages  add Chinese(simplified)
### 选择输入法引擎是IBus  
	选择keyboard input method system是 IBus
### install ibus需要的安装包 
```
  sudo apt-get install ibus ibus-clutter ibus-gtk ibus-gtk3 ibus-qt4
```
### 启动ibus框架（步骤一中第二步）
```
im-config -s ibus
```
说明：修改了输入法引擎需要重启X窗口系统
```
sudo systemctl restart lightdm.service
或者
reboot
```
### 安装ibus 拼音引擎
```
sudo apt install ibus-pinyin
```
### 配置输入法
	System Settings--> Text Entry--> input source to use  选择Chinese(Pinyin)(IBus)

### 测试

##注意：
	sublime 3对中文输入支持比较差！
