+++
title  = "vim常用插件收藏及简单说明"
isCJKLanguage = true
date = "2015-04-26T01:18:12+08:00"
+++

使用VIM也有四五年的时间了，期间自己搞过配置也用过不少别人的配置，但总是觉得哪里不对，很多配置我永远都用不到，但它的存在导致了VIM启动缓慢，也有速度很快，声称awesome的配置文件，实际使用起来
离自己的需求还差的很远，因此我想先在此把自己需要的功能整理规划一下，然后自己搞一套既能适应现在的习惯又能不浪费性能的配置。

下面是有需要的插件列表：

1. 插件管理工具 [gmarik/Vundle.vim](https://github.com/gmarik/Vundle.vim)

2. 查找工具
    - [scrooloose/nerdtree](https://github.com/scrooloose/nerdtree) + [vim-nerdtree-tabs](https://github.com/jistr/vim-nerdtree-tabs) 像Project一样展示目录树
    - ag 快速搜索目录中的文件
    - ctrlp + ctrlp-funky 搜索文件 + 搜索当前文件中的函数
    - [open_file_under_cursor](https://github.com/amix/open_file_under_cursor.vim) 打开位于光标下方的文件
    - Tagbar 显示tag列表
    - taglist 函数跳转（可以跨文件）
    - vim-multiple-cursors 多光标编辑
    - EasyMotion 快速查找

3. 语言相关
    - Python klen/php-mode 可认为是个Python IDE，功能强大，以一当十
    - PHP StanAngeloff/php.vim 语法高亮
    - JavaScript
        * jslint/jshint JS语法检查
        * pangloss/vim-javascript
        * jelera/vim-javascript-syntax
        * vim-coffe-script
    - html
        * Emmet.vim 神器，不怎么用，但还是留着吧，尝试着用一下
    - css
        * Better CSS Syntax for Vim
    - Markdown
        * vim-markdown 这个不能没有
        * Markdown syntax
    - JSON
        * JSON.vim

4. 补全工具
    - YouCompleteMe 补全神器，就是编译起来挺麻烦的
    - SuperTab 不用Tab的补全就不是真正的补全
    - ultisnips 代码片段，需要学习

5. 语法检查
    - syntastic

6. 代码整洁
    - vim-surround 方便的为选中的部分添加成对的符号
    - vim-comentary 方便的注释/取消注释
    - Tabular 符号对齐
    - EasyAlign
    - Indent Guides 代码缩进

7. 版本控制
    - fugitive.vim git wrapper

8. 系统工具
    - vimproc.vim vim内的交互式shell
    - <del>quickrun 在vim内执行脚本 用的时候一直觉得很不方便，因为执行结果会在一个新的window打开，还要再关闭现在已经配置好了一个快捷键，执行完回车即可关闭结果</del>

9. 杂项
    - airline 漂亮的状态栏
    - kien/rainbow_parentheses.vim 给成对的符号显示不同的颜色
    - Gundo 显示历史改动

能想到的插件就是这些，除了这些还需要加上一些我个人比较喜欢的定制

1. 相对行号，因为之前用NertCommenter，如果要注释掉下面的几行，就直接看左边的行号而不需要再像绝对行号的时候那么数了。

2. 把";"映射给":"，因为Normal模式下";"很不常用，而":"很常用，两个应该换一换。

3. Leader键用","，不知道当初是哪个配置把我带到这个沟里的，现在适应了就很难改了。
