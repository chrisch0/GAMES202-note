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
这样可见函数就从shading中拆分了出来，成为了Shadow mapping的理论基础

## PCF

对shading point在Shadow map上对应的像素周围的所有像素判断是否在阴影内，并对是否在阴影内的结果（0/1）进行平均。可以使用随机分布的采样点在Shadow map上进行采样

## PCSS



![PCSS](D:\Code\CG\202\GAMES202-note\03-note.assets\image-20210408172805192.png)