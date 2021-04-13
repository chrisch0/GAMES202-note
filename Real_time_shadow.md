## Shadow mapping的理论基础: 

式
$$
\int_{\Omega}f(x)g(x)dx \approx \frac{\int_{\Omega}f(x)dx}{\int_{\Omega}dx}\cdot\int_{\Omega}g(x)dx
$$

在满足以下两种情况其一时成立：

- $g(x)$的实际积分域很小（$\Omega$可能很大，但$g(x)$有值的部分很小）
- $g(x)$在积分域内的值变化不大（smooth）

再考虑渲染方程：
$$
L_{o}(p,\omega_{o})=\int_{\Omega^{+}}L_{i}(p,\omega_{i})f_{r}(p,\omega_{i},\omega_{o})\cos\theta_iV(p,\omega_i)d\omega_i
$$
在满足以下两种情况其一时：

* 光源是点光源或者平行光（$L_i(p,\omega_i)$的实际积分域很小）
* 光源是一个面光源，且表面是一个diffuse的表面时（$L_i(p,\omega_i)f_r(p,\omega_i,\omega_o)$在积分域内值的变化很小）

可以近似成：
$$
L_o(p,\omega_o)\approx \frac{\int_{\Omega^+}V(p,\omega_i)d\omega_i}{\int_{\Omega^+}d\omega_i} \cdot \int_{\Omega^+}L_i(p,\omega_i)f_r(p,\omega_i,\omega_o)\cos\theta_id\omega_i
$$
这样可见函数就从shading中拆分了出来，成为了Shadow mapping的理论基础。

## PCF (Percentage Closer Filtering)

对Shading point在Shadow map上对应的像素周围的所有像素判断是否在阴影内，并对是否在阴影内的结果（0或1）进行平均。可以使用随机分布的采样点在Shadow map上进行采样。

## PCSS (Percentage Closer Soft Shadows)

* PCSS的步骤

  1. 首先根据Receiver的深度，计算要进行Blocker search的区域的大小。

     ![image-20210413114216574](D:\Code\CG\202\GAMES202-note\Real_time_shadow.assets\image-20210413114216574.png) ![image-20210413112300981](D:\Code\CG\202\GAMES202-note\Real_time_shadow.assets\image-20210413112300981.png)
     $$
     W_{light}=\frac{LightSize}{FrustumWidth}
     $$

     $$
     W_{Search}=\frac{W_{light}(z_{Receiver}-z_{near})}{z_{Receiver}}
     $$

     

  2. 根据$W_{Search}$的大小，计算这块区域内Blocker的平均深度

     ```c
     float blockerSum = 0.0;
     int numBlockers = 0;
     for (int i = 0; i < BLOCKER_SEARCH_NUM_SAMPLES; ++i)
     {
         float shadowMapDepth = texture2D(shadowMap, receiverShadowMapUV.xy + poissonDisk[i] * searchWidth);
     
         if (shadowMapDepth < zReceiver)
         {
             blockerSum += shadowMapDepth;
             numBlockers += 1;
         }
     }
     float avgBlockerDepth = blockerSum / float(numBlockers);
     ```

 		3. 根据Blocker的平均深度，计算PCF过滤核的半径。

![image-20210413114426241](D:\Code\CG\202\GAMES202-note\Real_time_shadow.assets\image-20210413114426241.png)


$$
W_{penumbra}=\frac{z_{Receiver}-\bar{z}_{Blocker}}{\bar{z}_{Blocker}}\cdot W_{light}
$$
​		4. 根据计算出的$W_{penumbra}$进行PCF过滤，计算阴影系数。

* 数学描述：对可见度的卷积
  $$
  V(x)=\sum_{q\in\mathcal{N}(p)}{w(p,q) \cdot \chi^+[D_{SM}(q)-D_{scene}(x)]}
  $$

## VSSM (Variance Soft Shadow Mapping)

* VSSM是对PCSS在一定条件下的优化：

  对于PCSS来说，算法的核心就是对Shading point求解一个PCF过滤半径$r$，并计算在Shadow map上，Shading point的对应点周围半径$r$的范围内，被遮挡的像素比例。

  VSSM假设深度图上各像素的深度满足某单峰的概率分布，并且可以通过Mipmap或者SAT(Summed Area Tables)的方式快速得到某一区域深度的平均，以及深度平方的平均（在生成深度图的同时记录深度的平方）。

  1. 对于PCSS的第二步，就可以使用Chebyshev不等式，直接计算区域内被遮挡像素的比例的近似值（将$\leq$视作$\approx$）。

  $$
  P(x>t) \leq \frac{\sigma^2}{\sigma^2+(t-\mu)^2}(t>\mu)
  $$

  ​		其中，$t$为Shading point在Shadow中的深度，$P(x>t)$表示了Shadow map中，Shading point对应点周围深度大于$t$的像素的比例，即不被遮挡的比例。方差可由式$\sigma^2=E(x)-E(x^2)$求得。

  2. 对于PCSS的第一步，目的是求解Shadow map上被遮挡区域的平均深度（图中蓝色部分）。

     <img src="D:\Code\CG\202\GAMES202-note\Real_time_shadow.assets\image-20210413160757177.png" alt="image-20210413160757177" style="zoom:67%;" />

     设被遮挡区域的平均深度为$\bar{z}_{occ}$，未被遮挡部分的平均深度为$\bar{z}_{unocc}$，那么有
     $$
     \frac{N_1}{N}\bar{z}_{unocc}+\frac{N_2}{N}\bar{z}_{occ}=\bar{z}
     $$
     

     其中，$\frac{N_1}{N}=P(x>t), \frac{N_2}{N}=1-P(x>t)$，均可以使用Chebyshev不等式进行计算。并且，假设未被遮挡部分的深度有$z_{unocc}=t$（即假设了阴影的接收者为平面），即可计算出$\bar{z}_{occ}$。

* VSSM中查询Shadow map中区域的平均值的方法

  1. Mipmap

     <img src="D:\Code\CG\202\GAMES202-note\Real_time_shadow.assets\image-20210413163038160.png" alt="image-20210413163038160" style="zoom:80%;" />

     Mipmap的范围查询，存在不准的问题。

  2. SAT

     记录左上角所有像素的和。

     SAT查询：

     ![image-20210413163520570](D:\Code\CG\202\GAMES202-note\Real_time_shadow.assets\image-20210413163520570.png)

* VSSM缺陷

  Chebyshev不等式的局限：只能对概率密度函数只有一个峰值的分布进行估计，且只有在$t>\mu$时成立，可能会产生漏光的现象。

  <img src="D:\Code\CG\202\GAMES202-note\Real_time_shadow.assets\image-20210413164213019.png" alt="image-20210413164213019" style="zoom:80%;" />

## MSM (Moment Shadow Mapping)

* MSM通过更加精确地描述概率，改进VSSM的缺点：

  MSM使用更高阶的矩进行概率估计$x,x^2,x^3,x^4,...$，相比之下，VSSM只用了前两阶矩。前$m$阶矩可以表示有$\frac{m}{2}$个“台阶“”的概率累计分布函数。一般使用4阶矩。

  ![image-20210413165311916](D:\Code\CG\202\GAMES202-note\Real_time_shadow.assets\image-20210413165311916.png)

  

* 具体的流程与VSSM相似，生成深度图的同时记录$z,z^2,z^3,z^4$的值，生成对应的SAT，然后使用这些值还原概率累积分布函数，进行Blocker search和PCF。

## DFSS (Distance Field Soft Shadows)

* 向光源方向Ray March时，每次步进，SDF都会返回距离步进点最近的物体表面的距离，这个“安全”距离可以转化为一个“安全”角度，角度越大，意味着这个角度范围内，被遮挡的部分越少，可见性约接近于1；角度越小，越有可能被遮挡，可见性越接近于0。

  <img src="D:\Code\CG\202\GAMES202-note\Real_time_shadow.assets\image-20210413170631217.png" alt="image-20210413170631217" style="zoom:80%;" />

* 向光源方向Ray March时，取角度的最小值计算最终的可见度。

  <img src="D:\Code\CG\202\GAMES202-note\Real_time_shadow.assets\image-20210413171633765.png" alt="image-20210413171633765" style="zoom: 67%;" />

* 可见度计算：
  $$
  v=\min\{\frac{k\cdot SDF(p)}{p-o},1.0\}
  $$
  <img src="D:\Code\CG\202\GAMES202-note\Real_time_shadow.assets\image-20210413171909687.png" alt="image-20210413171909687"  />

