# shader graph

如果你想入门shader，shadergraph是个好主意。它提供了很多方法。

原理很简单，问题不大，就在普通的shader中，做一些修改。

节点：

channel节点：
    1. combine ： 把一堆数字组成一个vector
    2. split： 把vector转成一堆数值
    3. swizzle：通过一个式子，来把一些vector转成另外，比如(0,1,0,0) 通过(yyyy) 转成了（1,1,1,1)
    4. flip： 说是翻转一个vector，如果是（1, 0, 0) 会翻转成（-1, 0, 0)

artistic 节点
    adjustment: 调节颜色
        1. channel mixer: 一个乘法节点
        2. Contrast ： 跟0.5颜色的插值。1的就是原先颜色， 0就是0.5
        3. invert color： 这个才是翻转一个颜色。
        4. Replace Color Node： ？？？？
        5. Hue: ???
        6. Saturation : 调整颜色的饱和度。
        7. White Balance: ???
    blend： 混合颜色 Out = lerp(base, out, opacity)
        1. Blend：它里面有很多混合类型。
            Overwrite：就是简单的lerp。
            Multiply： Out = base * blend;
            很多。
    Filter：
        1. Dither：？？？
        2. Fade Transition：？？？
    Mask:
        1. Channel mask: 去掉一些channel
        2. Color mask：？？？
    Normal： 处理关于normal贴图的行为。
        1. Normal Blend： 融合两个normal贴图？？？
        2. ... ??? 这个后面再了解。
    Utility:
        1. Colorspace Conversion: 各种空间的color转化。比如RGB > Linear

Input 节点， 输入节点。
    1. Basic：一些基础数值，还有一些常量。
    2. Geometry: 一些shader的输入数值，比如顶点位置，uv，view dir，之类的。
    3. Gradient: ???
    4. High Definition Render pipeline: ???
    5. Lighting: 灯光相关的输入。
    6. Matrix： 矩阵相关的输入。
    7. Mesh Deformation:  ???
    8. PBR:???
    9. Scene: 场景相关的输入。
    10. Texture： 贴图相关的输入。

Math 节点。数学相关的节点。
    1. Basic： Add, Divide, Multiply, Power, Square Root(开根号), Subtract
    2. Advanced: Absolute, Exponential, Length,Log, Modulo, Negate (取反)， Normalize, Posterize(连续的颜色变化改成色块), Reciprocal(倒数)，Reciprocal Square Root（倒数开根号）
    3. Interpolation：Lerp, Inverse lerp, Smooth step.
    4. Matrix: Matrix Construction(创建一个矩阵), Matrix Determinant(???), Matrix Split(???), Matrix Transpose(转置矩阵)
    5. Range： Clamp, Fraction, Maximum, Minimum, One Minus, Random Range, Remap, Saturate(是clamp(0,1))
    6. Round: Ceiling, Floor, Round, Sign(0的sign = 0), Step（阶梯一样，不是0就是1，判断跟输入值的对比）, Truncate（截断，跟floor不一样，floor是向下取整, 它是往0方向靠）
    7. Trigonometry：(各种三角函数)Arccosine(cos), Arcsine(asin),Arctangent(atan(x)), Arctangent2(atan2(x,y)),Cosine(cos), Degree To Radian, Sin,Hyperbolic Cosine,Hyperbolic Sine, Hyperbolic CosineTangent
    8. Vector: 各种函数操作。Cross Product, Distance,  Dot, Fresnel Effect(菲涅尔效果), Projection(投影函数)，Reflection，Refract(折射)，Rejection(垂直到另外一个向量)，Rotate Around Axis(绕着轴转),Sphere Mask（球形遮罩，在球外面都是0），Transform（空间转化，从A空间切换到B空间）
    9. Wave： Noise Sine Wave（）, Sawtooth Wave(锯齿波), Square Wave(方波), Triangle Wave(三角波)

Procedural 程序化生成的。
    1. Noise 噪声图
        1. Gradient Noise:  梯度噪声, 有连续性，可以用来制作云朵之类的。
        2. Simple Noise: 普通噪声图
        3. Voronoi Noise：图案具有细胞状结构，适合模拟裂纹、细胞组织等。
    2. Shape 形状图
        1. Ellipse 椭圆
        2. Polygon 多边形
        3. rectangle 矩形
        4. roundPolygon 圆角多边形
        5. roundRectangle 圆角矩形
    3. CheckerBoard 棋盘的贴图

Utility 工具类
    1. Logic：一些逻辑运算和分支。All，And，Any，Branch，BranchOnInputConnection,Comparison, IsFrontFace, IsInfinity, IsNan, Nand, Not, Or
    2. High Definition Render Pipeline 渲染管线相关。 ？？？
    3. CustomFunction： 自定义函数， 可以自定义节点。
    4. Keyword: 开关函数
    5. Preview：可以预览某个阶段。
    6. Subgraph: 子节点。
    7. Subgraph Dropdown node

UV：对uv进行修改，实现一些效果的话，通过改变uv可以很好处理。
    1. Flipbook： ？？？ 
    2. Polar Coordinates： ？？？
    3. Radial Shear：？？？
    4. Rotate： 旋转, 可以让输入的uv发生旋转。
    5. Spherize： 球面化？？？
    6. Tiling And Offset：？？？
    7. Triplanar 
    8. Twirl 
    9. Parallax Mapping
    10. Parallax Occlusion Mapping

Block： shader的原本一些输入把。

可以使用ui来测试，它是最简单的一种。

shadergraph的实现原理： 
0. 可以点击资源的的inspector的上方有个 view generated code， 查看源码。
    1. 它的源码跟正常的shader一样，一般会包含多个LightMode，供各种的光照模型使用。
    2. 主要的shader可以是 sprite， lit， ulit这三种。
    2. 框架的代码在：SpriteUnlitPass.hlsl中, 它提供暴露了一个函数： SurfaceDescriptionFunction 用来处理 shadergraph中生成的代码。
    3. unit的框架是在 UnlitPass.hlsl中，lit的框架在：PBRForwardPass.hlsl
    4. 它的fragment输出结构体是特定的，不同的shader都不一样。
        1. lit 就是pbr的那些输出，它拿这些输出去算pbr。
        2. unlit就只有rgb
        3. sprite 除了rgb，还有一个a。
    
1. 有些节点也提供查看源码的方式，右键节点，view generated code。
