### 1. 基本概念

- **输入图像大小**：$i$
- **卷积核大小**：$k$
- **步幅**：$s$
- **填充**：$p$
- **输出图像大小**：$o$

### 2. 无padding，stride = 1

这是最常见的卷积方式，输出大小比输入小。

#### 第一种理解方式：
从一条边缘的角度看，每条边少掉的像素宽度为：卷积核大小的一半向下取整。从整张图片的角度看，这个少掉的量需要乘以2，考虑两侧的边。由于卷积核尺寸$k$一般是奇数，因此图片减小的尺寸为$k - 1$。得到输出尺寸公式：  
$$ o = i - k + 1 $$

#### 第二种理解方式：
以输入图片左上角为 $(0,0)$，右下角为 $(i-1, i-1)$。我们观察卷积核的最左上角像素在输入图片上的位置。当卷积核滑动到最左上角时，位置为 $(0,0)$；当卷积核滑动到最右下角时，位置为 $(i - k, i - k)$。区间 $[0, i-k]$ 中有 $i - k + 1$ 个整数。因此输出尺寸为：  
$$ o = i - k + 1 $$

示例：
- 输入大小 $i = 4$，卷积核大小 $k = 3$，输出大小为 $o = 4 - 3 + 1 = 2$

![无padding，stride=1](https://github.com/vdumoulin/conv_arithmetic/blob/master/gif/no_padding_no_strides.gif)

### 3. 无padding，stride > 1

当步幅大于1时，卷积核滑动的步长更大，输出尺寸减小的更为显著。

以输入图片左上角为 $(0,0)$，卷积核的最左上角像素在输入图像上的位置。在卷积核滑动过程中，当卷积核滑动到最左上角时，位置为 $(0,0)$；当卷积核滑动到最右下角时，位置为 $(i - k, i - k)$。我们临时用 $a$ 来表示 $i - k$，问题变成：  
$$ \text{for} \quad b = 0; \quad b \leq a; \quad b += s $$  
这个循环体内的操作会执行多少次？显然是  
$$ \left\lfloor \frac{a}{s} \right\rfloor + 1 $$ 次。因此输出尺寸为：  
$$ o = \left\lfloor \frac{i - k}{s} \right\rfloor + 1 $$

示例：
- 输入大小 $i = 5$，卷积核大小 $k = 3$，步幅 $s = 2$，输出大小 $o = \frac{5 - 3}{2} + 1 = 2$

![无padding，stride>1](https://github.com/vdumoulin/conv_arithmetic/blob/master/gif/no_padding_strides.gif)

### 4. 有padding，stride = 1

padding 就是指在输入图像的周围填充若干像素，使得输出相应的增大。最常见的填充方式是将输入的边缘用零值填充，也是 CNN 中默认使用的填充方式。  
输出大小为无padding的情况的输出大小 + 两倍的 padding，即：  
$$ o = i - k + 1 + 2p $$

#### 特殊情况：
1. **半填充 (same padding)**  
   在这种情况下，输入和输出大小相同。  
   卷积核的左上极限位置为卷积核中心在输入图像的 $(0,0)$ 位置。需要的填充大小为：  
   $$ p = \left\lfloor \frac{k}{2} \right\rfloor $$  
   或者  
   $$ p = \frac{k - 1}{2} $$  
   输出大小：  
   $$ o = i $$

   ![same padding](https://github.com/vdumoulin/conv_arithmetic/blob/master/gif/same_padding_no_strides.gif)

2. **全填充 (full padding)**  
   在这种情况下，卷积核的左上极限位置为卷积核的右下角在输入图像的 $(0,0)$ 位置。需要的填充大小为：  
   $$ p = k - 1 $$  
   输出大小：  
   $$ o = i + (k - 1) $$

   ![full padding](https://github.com/vdumoulin/conv_arithmetic/blob/master/gif/full_padding_no_strides.gif)

### 5. 有padding，stride > 1

当有padding且stride大于1时，卷积核的滑动和输出尺寸的计算方式类似。

考虑卷积核的最左上角像素在输入图片上的位置，左上角的极限位置为 $(-p, -p)$，右下角的极限位置为 $(i - k + p, i - k + p)$。此时，问题变成：  
$$ \text{for} \quad b = -p; \quad b \leq i - k + p; \quad b += s $$  
循环体内的操作会执行  
$$ \left\lfloor \frac{i - k + 2p}{s} \right\rfloor + 1 $$ 次，因此输出尺寸为：  
$$ o = \left\lfloor \frac{i - k + 2p}{s} \right\rfloor + 1 $$

示例：
- 输入大小 $i = 5$，卷积核大小 $k = 3$，填充 $p = 1$，步幅 $s = 2$，输出大小为  
  $$ o = \left\lfloor \frac{5 - 3 + 2}{2} \right\rfloor + 1 = 3 $$

![有padding，stride>1](https://github.com/vdumoulin/conv_arithmetic/blob/master/gif/padding_strides.gif)

示例：
- 输入大小 $i = 6$，卷积核大小 $k = 3$，填充 $p = 1$，步幅 $s = 2$，输出大小为  
  $$ o = \left\lfloor \frac{6 - 3 + 2}{2} \right\rfloor + 1 = 3 $$

![有padding，stride>1](https://github.com/vdumoulin/conv_arithmetic/blob/master/gif/padding_strides_odd.gif)

### 6. 附录

#### 其他特殊填充方式：
- **常数填充**：用常数值填充边缘像素。
- **镜像填充**：使用输入图像边缘的反向复制像素进行填充，像镜面一样复制。
- **复制填充**：将输入图像的边缘像素重复用于填充区域。
- **周期性填充 (Circular Padding)**：图像的边缘像素通过循环的方式与另一端像素相连接，类似周期性复制。

#### TensorFlow中的padding：
- `padding = valid`：无padding，$padding = 0$。
- `padding = same`：保持输出尺寸与输入相同。

### 7. GIF图片来源：
[https://github.com/vdumoulin/conv_arithmetic](https://github.com/vdumoulin/conv_arithmetic)