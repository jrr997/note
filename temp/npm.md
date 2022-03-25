# npm根据lockfile还是package.json来安装依赖？

npm安装时先会检测package-lock.json和package.json是否兼容，如果不兼容，以package.json为准。兼容的话会根据package-lock.json安装依赖。package-lock.json记录的是具体的版本。



参考资料：https://juejin.cn/post/7060844948316225572

