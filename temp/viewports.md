# viewports

- html标签的宽度等于视口的宽度。在PC中，视口的宽高等于浏览器窗口的宽高。在手机中视口比较复杂。

- 如何获取视口的宽高？ `document.documentElement.clientWidth` and `-Height`。注意document.documentElement指的是html标签。但当我们给html标签设置一个宽度，`document.documentElement.clientWidth`查出来的还是视口的宽度，与html标签的宽度无关。html标签的宽高可通过`document.documentElement.offsetWidth` and `-Height`获取。

- `window.innerWidth/innerHeight`与`document.documentElement.clientWidth/clientHeight`的区别？

  前者不包含scrollBar的宽高。这两组属性的出现与浏览器大战有关。
  
- 布局视口有最大值和最小值。最大值为10000px，最小值为1/10理想视口



## 理想视口

1. 什么是理想视口？

    It gives the ideal size of a web page on the device. Thus, the dimensions of the ideal viewport differ per device.

    ```
    设备像素比(dpr) ＝ 物理像素 / 设备独立像素
    ```

    物理像素：设备的真实像素，如iPhone6的横向物理像素为 750px

    设备独立像素：设备显示的像素。虽然iPhone6的横向有750个物理像素点，但横向最多只能显示375个像素。因为在iPhone6中，只看横向，2个物理像素显示一个像素的内容。

    retina屏幕未出现时(dpr = 1)，理想视口 = 设备物理像素。即在理想情况下，视觉视口 = 布局视口。

    当屏幕

2. 什么时候布局视口等于理想视口？

    ```
    width=device-width` and `initial-scale=1
    ```



放大倍数越大，视觉视口越小

不管布局视口现在的尺寸是多少，缩放系数都和理想视口有关

meta标签中scale = 2意思是放大倍数为理想视口的两倍



公式：

```
visual viewport width = ideal viewport width / zoom factor
zoom factor = ideal viewport width / visual viewport width
```





## 结论

用户看到的内容就是视觉视口
