## 后处理


## Volume：
    分为global的跟local的，
    global针对所有的camera
    local只针对进入collider的camera起作用

## performance
    bloom：中高

### bloom
bloom的原理是让画面中亮的地方更亮，
    1. 选取一个阈值（用户输入），然后shader把屏幕中的亮度大于阈值的点color输出到一个新图片上，然后blur一下，叠加到之前的图片上去。这样之前的图片亮的地方将会变得更亮。
    2. 问题来了
        1. 通过threshold来判定是否是亮的位置，通过intensity输出一个更亮的的贴图，叠加出来。
        2. 如何计算亮度，亮度的公式： 有多种方式可以获取，常用的就是 (r+g+b)/ 3, 或者是`L=0.299⋅R+0.587⋅G+0.114⋅B`, 这个是眼睛对颜色的敏感度来决定的。
        3. SCATTER,tint,clamp