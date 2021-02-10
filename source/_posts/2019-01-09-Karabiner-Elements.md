---
title: 神器 Karabiner Elements 配置
date: 2019-01-09 14:54:55
category: Other
tags: 
    - Karabiner Elements
---

Karabiner Elements，应该是 Mac 上必备的神器了，特别是使用 HHKB 的话。

## 简单配置 `Simple Modifications`

HHKB 的键位不用改，但是 Mac 原生的键盘需要改键位，打开 `Karabiner-Elements Preferences`，切换到 `Simple Modifications`，`Target Device` 选择 `Apple Internal Keybord / Trackpad(No manufacturer name)`。

键盘名字可能不一样，只要确定是要修改的 Mac 键盘就行，修改方式很简单，点击左下角 `Add item`，选择 `From key` 和 `To key`，`From key` 为要修改的键，`To key` 为要修改成什么键。

Mac 原生键盘和 HHKB 统一的话使用以下配置，也就是左边 `control` 和 `caps lock` 交换位置，`delete` 和 `\` 交换位置。

![image](/images/屏幕快照 2019-01-09 15.02.22.png)

## 使用现成的规则

```
Complex Modifications -> Add rule -> Import more rules from the Internet(open a web browser)
```

搜索 Emac，添加 Emacs key bindings，添加完之后对应的规则为：

 - Key Bindings (C-x key strokes)
    ```
    C-x C-c Quit application (post command-q)
    C-x C-f Open file (post command-o)
    C-x C-s Save file (post command-s)
    ```

 - Key Bindings (control+keys)
    ```
    control+d   forward delete
    control+h   delete
    control+i   tab
    control+[   escape
    control+m   return
    control+bfnp    arrow keys
    control+v   page down
    control+a (Microsoft Office)    home
    control+e (Microsoft Office)    end
    ```

 - Key Bindings (option+keys)
    ```
    option+v    page up
    option+bf   option+arrow keys
    option+d    option+forward delete
    ```

 - Bash Style Key Bindings
    ```
    control+w   Delete the word behind point (option+delete)
    control+u   Delete backward from point to the beginning of the line.
    ```


## 自定义

添加完规则后有一些不完善的地方，例如，我喜欢用 `Control + j/l` 作为跳单词的快捷键，也就是代替 `opt` 加左右方向键，另外，设置完快捷键后 hhkb 在 vim 上方向组合键部分无效。

修改规则的时候只看比较重要的地方，我们要修改地方都是在 `rules` 下，`rules` 是一个数组，每条规则是一个 `manipulators`，`manipulators` 主要结构如下：

```
https://pqrs.org/osx/karabiner/json.html#complex_modifications-manipulator-definition
```

`Move Caret to Previous Word`
`Move Caret to Next Word`



