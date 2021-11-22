不知道大家用Typora写笔记的时候，有没有觉得加完代码块后，去选择语言很麻烦？像下面这样：

![image-20210825213816603](怎么给Typora的代码块设置默认语言？.assets/image-20210825213816603.png) 



经过一番探索后（网上还真没搜到，哭。。。），发现使用ahk脚本可以解决这个问题。

操作如下：

1、链接下载ahk，https://autohotkey.com/download/ahk-install.exe

安装完成后，就可以直接新建ahk脚本了（桌面右键，或者txt改后缀都行）

```java
#IfWinActive ahk_exe Typora.exe
{
    ; Ctrl+Alt+K javaCode    
    ; crtl  是  ^      alt   是   !    k  是  k键
    ^!k::addCodeJava()
}
addCodeJava(){
Send,{Asc 096}
Send,{Asc 096}
Send,{Asc 096}
Send,java
Send,{Enter}
Send,{Enter}
Return
}
```

把上面内容复制到文件中。保存，运行即可。

我使用的是typora默认代码块的快捷键 Ctrl+Alt+K 。也可根据需要自行替换。

搞定。

```java

```



2、原理也很简单：

就是我们使用快捷键的时候，脚本帮我们写了个  ````java+Enter`，也就是代码块+java的语法。其他语言都类似，只要把脚本中的 java 换成需要的语言就可以了。



> 最后，我发现使用微软拼音的时候会失效，可能是对应的asc码不对了。我暂时就不研究了。如果其他输入法也有问题，大伙可以摸索下怎么输出 ` 就好了。
>
> 我是用的搜狗输入法，测了没问题。





3、对语法感兴趣的可以看看：

Send,{Asc 096} 表示  输出  `   

asc码的096就是 `

Send,java 表示输出  java
Send,{Enter} 等于是  敲了  回车键。



还可以编写其他脚本，比如给我们的字体加颜色。

```java
; Typora
; 快捷增加字体颜色
; SendInput {Text} 解决中文输入法问题

#IfWinActive ahk_exe Typora.exe
{
    ; Ctrl+Alt+O 橙色
    ^!o::addFontColor("orange")

    ; Ctrl+Alt+R 红色
    ^!r::addFontColor("red")
     
    ; Ctrl+Alt+B 浅蓝色
    ^!b::addFontColor("cornflowerblue")
}

; 快捷增加字体颜色
addFontColor(color){
    clipboard := "" ; 清空剪切板
    Send {ctrl down}c{ctrl up} ; 复制
    SendInput {TEXT}<font color='%color%'>
    SendInput {ctrl down}v{ctrl up} ; 粘贴
    If(clipboard = ""){
        SendInput {TEXT}</font> ; Typora 在这不会自动补充
    }else{
        SendInput {TEXT}</ ; Typora中自动补全标签
    }
}
```

选择要加颜色的字，输入快捷键Ctrl+Alt+O 橙色。

<font color='orange'>这样使用就可以方便的给字体加颜色了。</font>