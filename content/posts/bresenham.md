---
math: true
---
讨论斜率小于1的情况，测试x轴为单位变化，而y轴则小于单位变化(1px), 
$$
\begin{align*}
 y &= m(x_{k} + 1) + b \\
 d_{lower} &= y -y_{k} = m(x_{k} + 1) + b -y_{k} \\
 d_{upper} &= (y_{k} + 1) - y = y_{k} + 1 - m(x_{k} + 1) - b
\end{align*}
\\
\begin{align*}
   d_{lower} - d_{upper} &= m(x_{k} + 1) + b - y_{k} - (y_{k} + 1 - m(x_{k} + 1) - b) \\
                         &= 2m(x_{k} + 1) - 2y_{k} + 2b - 1 \\
\end{align*}
$$
主要用来确定是$y_{k}$还是在$y_{k+1}$上绘制。  

![](../images/graphics_lower_upper_in_y_axis.png)

设$\Delta y$和$\Delta x$为输入的两个端点的垂直和水平偏移量，令$m=\Delta y / \Delta x$, 将决策参数定义为：
$$
\begin{align*}
p_{k} &= \Delta x(d_{lower} - d_{upper}) \\
      &= 2\Delta y \cdot x_{k} - 2\Delta x \cdot y_{k} + c
\end{align*}
$$
为什么这么定义决策公式，个人认为是为了去掉除法运算，因为$m=\Delta y / \Delta x$，减少误差，所以故意多乘以一个$\Delta x$。

例子中$p_{k}$的符号与$d_{lower}-d_{upper}$的符号相同(因为例子中$\Delta x > 0$)，参数c是一个常量，简单公式换算就可得到，其值为$2\Delta y+\Delta x(2b-1)$, 它与像素位置无关，且会在循环计算$p_{k}$时被消除。假如$y_{k}$的像素比$y_{k+1}$的像素更接近于线段(即$d_{lower} < d_{upper}$)，那么参数$p_{k}$是负的。此时绘制下面的像素，反之，绘制上面的像素。
可以利用递增整数运算得到后继的决策参数值。可以减少乘法运算：
$$
\begin{align*}
p_{k+1} &= 2\Delta y \cdot x_{k+1} - 2\Delta x \cdot y_{k+1} + c \\
p_{k+1} - p_{k} &= 2\Delta y(x_{k+1} - x_{k}) - 2\Delta x(y_{k+1} - y_{k})
\end{align*}
$$
但是$x_{k+1} = x_{k} + 1$, 因而得到
$$
p_{k+1} = p_{k} + 2\Delta y - 2\Delta x(y_{k+1} - y_{k})
$$
其中，$y_{k+1}-y_{k}$取0或1，取决于参数$p_{k}$的符号。这是因为$p_{k}$决定了下一个点的绘制位置(查看$p_{k}公式$)

起始像素位置$(x_{0}, y_{0})$的第一个参数$p_{0}$通过等式(3.14)及$m=\Delta y / \Delta x$计算得出：
$$
p_{0} = 2\Delta y - \Delta x
$$

下面是详细描述$|m| < 1$时的Bresenham的画线算法：
1. 输入线段的两个端点，并将左端点存储在$(x_{0}, y_{0})$中；
2. 将$(x_{0}, y_{0})$装入帧缓冲，画出第一个点；
3. 计算常量$\Delta x$, $\Delta y$，$2\Delta y$和$2\Delta y - 2\Delta x$，并得到决策参数第一个值:$p_{0} = 2\Delta y - \Delta x$；
4. 从$k=0$开始，在沿线路径的每个$x_{k}$处，进行下列检测:
   * 如果$p_{k} < 0$，下一个要绘制的点时$(x_{k}+1, y_{k})$，并且${p_{k+1} = p_{k} + 2\Delta y}$；
   * 否则，下一个要绘制的点是$(x_{k}+1, y_{k}+1)$，并且$p_{k+1} = p_{k} + 2\Delta y - 2\Delta x$；
5. 重复步骤4，共$\Delta x -1$次。

下面是$0<m<1.0$的画线算法的实现：

```cpp
#include <stdlib.h>
#include <math.h>

void lineBres(int x0, int y0, int xEnd, int yEnd) {
      int dx = fabs(xEnd - x0), dy = fabs(yEnd - y0);
      int p = 2 * dy - dx;
      int twoDy = 2 * dy;, twoDyMinusDx = 2 * (dy - dx);
      int x, y;
      if (x0 > xEnd) {
            x = xEnd;
            y = yEnd;
            xEnd = x0;
      } else {
            x = x0;
            y = y0;
      }
      setPixel (x, y);
      while(x < xEnd) {
            x++;
            if (p < 0) 
                  p+=twoDy;
            else {
                  y++;
                  p += twoDyMinusDx;
            }
            setPixel (x, y);
      }
}
```
