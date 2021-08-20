---
layout: post
title: OpenGL ES 读书笔记
date: 2017-12-20 18:20:51
tags:  Android, OpenGL, OpenGL ES, GlSurfaceView
---
# OpenGL ES 读书笔记

[TOC]

> OpenGL（Open Graphics Library）是指定义了一个跨编程语言、跨平台的编程接口规格的专业的图形程序接口。它用于三维图像（二维的亦可），是一个功能强大，调用方便的底层图形库。
>
> OpenGL ES (OpenGL for Embedded Systems)是 OpenGL的子集，针对手机、PDA和游戏主机等嵌入式设备而设计的。OpenGL ES 是从 OpenGL 裁剪的定制而来的，去除了glBegin/glEnd，四边形（GL_QUADS）、多边形（GL_POLYGONS）等复杂图元等许多非绝对必要的特性。
>

# 一、 基础概念

- **顶点**：
一个顶点就是一个代表几何对象的拐角的点，这个点有很多附加属性；最重要的属性就是位置，它代表了这个顶点在空间中的定位。

- **attribute**：属性

- **vec4**：  
    具有四个分量的向量Vector。

- **uniform**

- **varying**： 是一个特殊的变量，他把给它的那些值进行混合，并把这些混合后的值发送给片段着色器。

- **mat4**：具有四个量的矩阵；

- **归一化设备坐标**：在openGL里，我们要涫染的一切物体都要映射到x轴和y轴上一1， 1的范围内，对z轴也是一样的。这个范围内的坐标 被称为归一化设备坐标


# 二、坐标系

在openGL里，我们要涫染的一切物体都要映射到x轴和y轴上一1， 1的范围内，对z轴也是一样的。这个范围内的坐标 被称为归一化设备坐标，其独立于屏幕实际的尺寸或 形状。

不幸的是．因为它们独立于实际的屏幕尺寸，如 果直接使用它们．我们就会遇到问题．例如在横屏模 式情况下被压扁的桌子G     假设实际的设备分辨率以像素为单位是1280×720． 这在新的Andro记设备上是一个常用的分辨率。为了 使讨论更加容易，让我们也暂时假定OpenGL占用整 个显示屏。     如果设备是在竖屏模式下，那么-1，1』的范围对 应1280像素高．却只有720像素宽。图像会在X轴显 得扁平，如果在横屏模式，同样的问题也会发生在y 轴上。     归一化设备坐标假定坐标空间是一个正方形．

# 三、着色器语言

在 OpenGL ES里，只能绘制点、直线以及三角形。

当我们定义三角形的时候，我们总是以逆时针的顺序排列顶点；这称为卷曲顺序（windingorder），因为在任何地方都使用这种一致的卷曲顺序，可以优化性能：使用卷曲顺序可以指出一个三角形属于任何给定物体的前面或者后面，OpenGL 可忽略那些无论如何都无法被看到的后面的三角形。

# 四、着色器

Open GL 绘制流程

![image](https://github.com/davyjoneswang/static-res/blob/master/%E7%AE%A1%E9%81%93.png?raw=true)

1·顶点着色器（vertexshader）生成每个顶点的最终位置，**针对每个顶点，它都会执行一次**；一旦最终位置确定了，OpenGL 就可以把这些可见顶点的集合组装成点、直线以及三角形。

2，片段着色器（fragmentshader）为组成点、直线或者三角形的每个片段生成最终的颜色。片段着色器的主要目的就是告诉GPU每个片段的最终颜色是什么。针对每个图元，它都会执行一次，一个片段是一个小的、单一颜色的长方形区域，类似于计算机屏幕上的一个像素。


## 顶点着色器

```
attribute vec4 a_Position;

void main()
{
    gl_Position = a_Position;
}
```

顶点着色器里必须给gl_Position赋值，来保存顶点的坐标。

## 片段着色器

什么是片段着色器？片段着色器是怎么产生的？

OpenGL通过光栅化把每个点、直线及三角形分解成大量的小片段，他们可以映射到移动设备显示屏的像素上，从而生成一幅图像。这些片段类似显示屏上的像素，每一个都包含单一的纯色。为了表示颜色，每个片段都有四个分量：其中红色、绿色、蓝色、用来表示颜色，alpha用来表示透明度。


```
precision mediump float;

uniform vec4 u_Color;

void main()
{
    gl_FragColor = v_Color;
}
```

第一句用来定义浮点数据类型的精度。可选为lowp,mediump,highp.
顶点着色器的默认精度为highp。无需定义。

uniform： 我们使用uniform 来保存颜色**。uniform相对于atrribute的不同，atrribute每个顶点都要设置一个，而uniform会让每个顶点都是用同一个值，**除非我们再次改变他的值。
在片段着色器总，vec4代表了颜色的四个分量。而在顶点着色器中,vec4代表x ,y, z, w 四个坐标分量。


## 着色器的使用
着色器代码就是一段文本程序。需要经过一定处理才能使用，就像我们写的程序，需要编译，链接、生成可执行文件一样。着色器也需要经过类似的步骤。

这个过程有着固定的步骤。
1. 生成作色器。

```
final int shaderObjectId = glCreateShader(type);
if (shaderObjectId == 0) {
    return 0;
}

```

2. 上传着色器代码。

```
// Pass in the shader source.
 glShaderSource(shaderObjectId, shaderCode);
```

3. 编译载着色器代码

```
// Compile the shader.
  glCompileShader(shaderObjectId);
```

4. 获取并验证编译结果

```
// 获取编译状态
final int[] compileStatus = new int[1];
glGetShaderiv(shaderObjectId, GL_COMPILE_STATUS, compileStatus, 0);
// 打印着色器对象信息
Log.v(TAG, "Results of compiling source:" + "\n" + shaderCode + "\n:"
                + glGetShaderInfoLog(shaderObjectId));

// 验证编译结果
if (compileStatus[0] == 0) {
    // 编译失败，删除对象
    glDeleteShader(shaderObjectId);

    Log.w(TAG, "Compilation of shader failed.");
    return 0;
}
```


## Open GL程序
有了着色器，Open GL还不能直接工作。就像我们有了一些库的，必须有一个可执行的程序才能工作，同样，Open GL 也需要一个Program.

我们需要一个程序，把顶点着色器和片段着色器链接起来才能一起工作。步骤如下：

1. 创建程序对象。

```
// Create a new program object.
final int programObjectId = glCreateProgram();

if (programObjectId == 0) {
    Log.w(TAG, "Could not create new program");
    return 0;
}
```

2. 附着着色器。
有了程序对象，我们需要加载着色器对象到程序中。包括顶点着色器和片段着色器。

```
   // Attach the vertex shader to the program.
   glAttachShader(programObjectId, vertexShaderId);

   // Attach the fragment shader to the program.
   glAttachShader(programObjectId, fragmentShaderId);
```

3. 链接程序

```
// Link the two shaders together into a program.
glLinkProgram(programObjectId);

```

4.  获取并验证链接过程状态

和着色器类似，我们也需要验证链接过程中没有出错，以确保使用时不会发生错误。

```
// 获取链接状态
final int[] linkStatus = new int[1];
        glGetProgramiv(programObjectId, GL_LINK_STATUS, linkStatus, 0);
 // Print the program info log to the Android log output.
            Log.v(TAG, "Results of linking program:\n"
                + glGetProgramInfoLog(programObjectId));		

        // 验证链接状态
        if (linkStatus[0] == 0) {
            glDeleteProgram(programObjectId);
             Log.w(TAG, "Linking of program failed.");
            return 0;
        }
```


## 程序的使用


### 验证
在使用程序之前，我们还需要验证程序对象，看看这个程序对于当前的Open GL 状态是不是有效。根据Open GL ES 2.0 的文档，它给OpenGL提供了一种方法让我们知道为什么当前的程序可能是低效率的、无法运行，等等、


```
glValidateProgram(programObjectId);

final int[] validateStatus = new int[1];
glGetProgramiv(programObjectId, GL_VALIDATE_STATUS, validateStatus, 0);
Log.v(TAG, "Results of validating program: " + validateStatus[0]
            + "\nLog:" + glGetProgramInfoLog(programObjectId));
return validateStatus[0] != 0

```

调用glValidateProgram(programObjectId)来验证程序的状态。然后使用GL_VALIDATE_STATUS作为参数来调用glGetProgramiv，获取检查结果，并打印一些程序信息。

### 使用

```
glUseProgram(program);
```
首先，要使用glUseProgram告诉OpenGL,我们使用这个程序绘制.

现在我们可以真正使用这个程序了。

我们通过给着色器对象中的变量赋值。来让OpenGL绘制我们想要的东西。

> OpenGL将着色器编译为一个程序的时候。他实际上用一个位置编号把片段着色器中定义的每一个uniform都关联起来.我们通过位置编号来给着色器发送数据。像uniform一样，在使用属性之前我们也要获得它们的位置。我们可以让 OpenGL 自动给这些属性分配位置编号，或者在着色器被链接到一起之前，可以通过调用glBindAttribLocation（）由我们自己给它们分配位置编号。让OpenGL自动分配这些属性位置，使代码更容易管理。


1. 要想给着色器对象中定义的变量赋值。我们需要获取到这些变量在程序中的位置。（这些变量的位置在程序编译完后，就固定了。）


2. 获取uniform位置。

```
uniformLocation = glGetUniformLocation(program, “uniform变量的名字”);
```
在上面参数就是u_Color

3. 获取attribute位置。
```
attributLocation = glGetUniformLocation(program, “attribut变量的名字”);
```
在上面参数就是a_Position.

4. 给变量赋值。关联顶点数据，也就是给着色器赋值。

我们的数据都是放到FloatBuffer中的。

```
vertexData.position(0);
//关联数据
glVertexAttribPointer(aPositionLocation, POSITION_COMPONENT_COUNT, GL_FLOAT,
false, STRIDE, vertexData);
//使能数据
glEnableVertexAttribArray(aPositionLocation);
```

> OpenGL的坐标到屏幕映射。OpenGL会把坐标映射到【-1，1】的范围内，也就是屏幕会被映射为-1，1、 1，1、 -1，-1、 1，-1. 这第四个点为屏幕四个角的坐标。0，0为屏幕中心。

5. 绘制

刚才说过，OpenGL可以绘制点，线，三角形。

在每次绘制前，我们要给片段着色器赋值。

```
//先给三角形赋值颜色。
glUniform4f(uColorLocation, 1.0f, 1.0f, 1.0f, 1.0f);
//绘制三角形。
glDrawArrays(GL_TRIANGLE, 0, 6); 参数为顶点数据在FloatBuffer数据的起始索引和长度。
//绘制线
glDrawArrays(GL_LINES, 6, 2); 参数为顶点数据在FloatBuffer数据的起始索引和长度。
//绘制点
glDrawArrays(GL_POINTS, 9, 1); 参数为顶点数据在FloatBuffer数据的起始索引和长度。
```

# 五、颜色和着色

## 三角形扇
第一个点与后面的每两个点构成一个三角形。
## 平滑着色
线段，三角形之间片段颜色采用顶线之间线性着色。
varying： 通过varying变量产生平滑着色。

顶点着色器定义varying变量来生成着色， 片段着色器定定义相同的varying变量，来接收顶点着色器的值。

如果一个片段属于一条直线，那么 OpenGL就会用构成那条直线的两个顶点计算其混合后的颜色。如果那个片段属于一个三角形，那 OpenGL 就会用构成那个三角形的三个顶点计算其混合后的颜色。

# 六、OpenGL三维

## 三维化原理

二维三维化原理：
就是透视。在一个想象中的消失点处，把并行线段聚合在一起，从而创建出三维的幻像。
[透视原理](https://baike.baidu.com/item/%E9%80%8F%E8%A7%86%E5%8E%9F%E7%90%86)
近大远小

## 从着色器到屏幕的坐标变换

一个顶点着色器上的顶gl_Postion点最终变换为屏幕坐标的过程
```
graph LR
gl_Postion-->透视化除法
透视化除法-->归一化坐标
归一化坐标-->窗口坐标
```
两个变换，三种不同的坐标空间。

### 裁剪

当顶点着色器把一个值写到gl_Position的时候，OpenGL会做裁剪。 OpenGL使用透视除法将坐标裁剪到裁剪空间中。

## 透视除法

针对每个坐标，OpenGL将想x,y,z分量除以w. 这样都会位于
-w,w范围内。这样之后，较远的物体会被移动到距离渲染中心更近的地方。这个点就像是一个消失点。

（0，0，0）就是归一化坐标的渲染区域的中心。

因为透视除法，裁剪空间中的坐标被称为同质化坐标。不同的点可能会映射到相同的点。

### 视口变换

OpenGL把归一化坐标的x,y 分量映射到屏幕的一个区域，被称为视口。
glViewport就是设置视口的。


> 归一化坐标默认使用的是左手坐标系。

## 透视投影
在前面章节中，使用正交投影处理屏幕的宽高问题，它通过调整显示区域的宽高使之变换为归一化坐标。在使用正交投影时，我们假定一个包围整个场景的立方体。

### 视椎体

使用透视投影，立方体的线延长后就会回合到一点，成为一个椎体，叫做视椎体。

视椎体只是一个立方体，它的近端比远端大，看起来就是一个金字塔。两端的大小差距越大，观察范围越广，能看的也就越多。

视椎体有大端和小端。
焦点：金字塔的顶。
焦距：焦点到小端的距离。影响大端和小端的比例。

### 投影矩阵


### 模型矩阵
使用模型矩阵来平移场景中的物体。因为默认z位置在0， 需要使用模型矩阵沿着负轴移动。

矩阵操作： OpenGL的Matrix提供了矩阵操作的方法，可以方便对矩阵进行相乘，平移，旋转等操作。

# 七、纹理
纹理：是图像，图片，上传到OpenGL。叫纹理
POT纹理: 图片的宽高都是2的幂的纹理。

## 纹理坐标

每个二维的纹理都有其自己的坐标空间，其范围是从个拐角的（0,0）到另一个拐角的（11）。按照惯例，一个维度叫做 s，而另一个称为T。当我们想要把一个纹理应用于个三角形或一组三角形的时候，我们要为每个顶点指定一组 ST 纹理坐标，以便 OpenGL 知道需要用那个纹理的哪个部分画到每个三角形上。这些纹理坐标有时也会被称为 UV 纹理坐标，如图 7-3 所示。

## 加载纹理。

1. 创建纹理

```
 final int[] textureObjectIds = new int[1];
 //数量，存储数组、偏移
 glGenTextures(1, textureObjectIds, 0);
```
2. 删除纹理

```
//数量，存储数组、偏移
glDeleteTextures(1, textureObjectIds, 0);
```

3. 绑定纹理对象到OpenGL

```
glBindTexture(GL_TEXTURE_2D, textureObjectIds[0]);
```
GL_TEXTURE_2D 告诉OpenGL 将纹理看做是一个二维纹理，
textureObjectIds[0] 具体的纹理对象

4. 纹理过滤
纹理过滤指纹理在被放大和缩小时，OPenGL如何处理纹理。

最近邻过滤：OPenG为每个片段选择最近的纹理。放大时，会有细节丢失。产生锯齿。
双线性过滤：OPenG使用双线性插值平滑像素之间的过滤。使用四个邻接的纹理，并用一个线性插值算法做插值。双线性指的是它是沿着两个维度插值的。

MIP贴图
双线性过滤：因为每一帧都要选择不同纹理元素，这会引起噪音和移动中物体的闪数。
MIP贴图：MIP贴图可以用来生成一组优化的的不同大小的纹理。 当生成纹理时，OpenGL会使用所有的纹理元素生成每个级别的纹理，在渲染时，OpenGL会根据每个片段的纹理元素数量为每个片段选择最合适的级别。

三线性过滤:
最邻近的MIP贴图级别也要插值。


```
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```
glTexParameteri设定过滤器
制定缩小模式时，使用三线性过滤
制定放大模式时，使用双线性过滤


## 加载纹理到OpenGL

```
// 加载纹理到OpenGL
texImage2D(GL_TEXTURE_2D, 0, bitmap, 0);

// 生成贴图
glGenerateMipmap(GL_TEXTURE_2D);

// 回收位图.
bitmap.recycle();

// 解绑纹理
glBindTexture(GL_TEXTURE_2D, 0);

```
至此，纹理贴图已经生成。

##  纹理顶点着色器


```
uniform mat4 u_Matrix;

attribute vec4 a_Position;
attribute vec2 a_TextureCoordinates;

varying vec2 v_TextureCoordinates;

void main()
{
    v_TextureCoordinates = a_TextureCoordinates;
    gl_Position = u_Matrix * a_Position;
}
```

a_TextureCoordinates 纹理坐标，有两个分量，S和T。

v_TextureCoordinates 我们将上面的坐标传给顶点着色器做插值。


##  纹理片段着色器


```
precision mediump float;

uniform sampler2D u_TextureUnit;
varying vec2 v_TextureCoordinates;

void main()
{
    gl_FragColor = texture2D(u_TextureUnit, v_TextureCoordinates);
}
```

OPenGL为每个片段调用片段着色器，并且用v_TextureCoordinates接受纹理坐标。

sampler2D u_TextureUnit: 片段着色器通过sampler2D接收实际的纹理数据

被插值的纹理坐标和纹理数据被传递给着色器函数 texture2D (），它会读入纹理中那个特定坐标处的颜色值。接着通过把结果赋值给 glFragColor 设置片段的颜色。



纹理绘制

当我们在 OpenGL 里使用纹理进行绘制时，我们不需要直接给着色器传递纹理。相反，我们使用纹理单元（textureunit）保存那个纹理。之所以这样做，是因为一个 GPU 只能同时绘制数量有限的纹理。它使用这些纹理单元表示当前正在被绘制的活动的纹理。


```
 //激活纹理单元0
glActiveTexture(GL_TEXTURE0);

 // 将纹理对象绑定纹理当前单元
glBindTexture(GL_TEXTURE_2D, textureId);

// tell the texture uniform sampler to use this texture in the shader by
// telling it to read from texture unit 0.
// 告诉GL使用纹理单元1的纹理给片段着色器的uTextureUnitLocatio赋值。
glUniform1i(uTextureUnitLocation, 0);
```

纹理坐标 （ST）UV坐标：


![image](https://note.youdao.com/yws/api/personal/file/WEBbac1fbac0f48726d29f7dd1a2fa3dc78?method=getImage&version=1494&cstk=aNW9Q6WS)


图像坐标：

![image](https://note.youdao.com/yws/api/personal/file/WEB72e27e72a7ef593fafa3651359bd5d99?method=getImage&version=1495&cstk=aNW9Q6WS)


# 八、构建物体

## 合并三角形带和三角形扇

> **三角形带**： 一个三角形带的前三个顶点定义了第一个三角形。这之后的每个额外的顶点与前两个顶点定义了另外的三角形。
>
> **三角形扇**：
> 一个三角形扇的前三个顶点定义了第一个三角形。这之后的每两个顶点与第一个顶点定义了另外的三角形。

## 添加几何图形

## 添加物体构建器


### 触摸反馈
我们需要将触摸事件交给渲染器处理。
Android中使用setOntouchListener接受事件处理。

坐标转换。
Android中事件坐标为坐左上为0,0. 右下角为w，h。
OpenGL坐标为归一化坐标。需要把Android坐标转换为归一化坐标：


```
final float x = ((float)event.getX() / (float)v.getWidth) * 2 - 1.
final float y = -(((float)event.getY() / (float)v.getHeight)) * 2 - 1).
```

### 将事件交给渲染器处理

Android的事件处理在主线程。而OpengGL在子线程，使用下面的方法将事件传递给Android的主线程。

```
glSurfaceView.queueEvent(new Runnable);
```
