## path

`path.resolve()`：拼接路径

`path.join()`：拼接路径

resolve比join智能，resolve能自动处理分隔符，我们不用担心操作系统不能识别分隔符的情况

`path.dirname()`：获取目录名

`path.basename()`：获取文件名

`path.extname()`：获取文件名后缀



## fs

1. fs对文件进行操作，主要分为三种操作：

- 同步操作,例如`fs.statSync()`
- 异步操作，例如`fs.stat()`
- Promise方式，例如`fs.promises.stat()`

2. 文件描述符，一些方法中可以用文件描述符来找到指定的文件。
3. 常用方法
`fs.readFile()`：读取文件
`fs.writeFlie()`：写入文件
`fs.mkdir()`：创建文件
`fs.readdir()`：读取文件夹的所有文件，返回一个包含所有文件的数组
`fs.rename()`：重命名



## events

发射事件