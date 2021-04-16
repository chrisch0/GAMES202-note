## IBL (Image-Based Lighting)

* 理论基础

  由于是环境光照，不考虑可见性，渲染方程为：
  $$
  L_o(p,\omega_o)=\int_{\Omega^+}L_i(p,\omega_i)f_r(p,\omega_i,\omega_o)\cos\theta_id\omega_i
  $$
  对于glossy的BRDF，它的对应的积分域就很小；对于diffuse的BRDF，它的值就相对smooth，就可以将渲染方程转化为以下的近似：
  $$
  L_o(p,\omega_o) \approx \frac{\int_{\Omega^+}L_i(p,\omega_i)d\omega_i}{\int_{\Omega^+}d\omega_i} \cdot \int_{\Omega^+}f_r(p,\omega_i,\omega_o)\cos\theta_id\omega_i
  $$
  因此环境光照可以转化为$\frac{\int_{\Omega^+}L_i(p,\omega_i)d\omega_i}{\int_{\Omega^+}d\omega_i}$和$\int_{\Omega^+}f_r(p,\omega_i,\omega_o)\cos\theta_id\omega_i$两项分别计算。

* $\frac{\int_{\Omega^+}L_i(p,\omega_i)d\omega_i}{\int_{\Omega^+}d\omega_i}$的计算

  在预处理阶段，对环境纹理进行Prefiltering。

  ![image-20210413181433913](D:\Code\CG\202\GAMES202-note\Environment_mapping.assets\image-20210413181433913.png)

  然后在镜面反射方向查询对应的环境光强度

  ![image-20210413181612024](D:\Code\CG\202\GAMES202-note\Environment_mapping.assets\image-20210413181612024.png)

  

  

