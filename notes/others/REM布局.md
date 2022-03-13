# REM布局

如何把css中的px单位替换为rem单位？(px2rem的原理)

1. nodejs的fs模块读取.css文件: .css ->  javascript字符串
2. 引入css库，利用css.parse(): 字符串 -> css AST
3. 遍历AST，把px替换成rem
4. css.stringify(AST) -> 字符串

这里可以不生成AST,直接正则匹配字符串中的px。