## IBL (Image-Based Lighting)

* 理论基础

  由于是环境光照，不考虑可见性，渲染方程为：
  $$
  L_o(p,\omega_o)=\int_{\Omega^+}L_i(p,\omega_i)f_r(p,\omega_i,\omega_o)\cos\theta_id\omega_i
  $$
  对于glossy的BRDF，它的对应的积分域就很小；对于diffuse的BRDF，它的值就相对smooth，就可以将渲染方程转化为以下的近似：
  $$
  L_o(p,\omega_o) \approx \frac{\int_{\Omega_{f_r}}L_i(p,\omega_i)d\omega_i}{\int_{\Omega_{f_r}}d\omega_i} \cdot \int_{\Omega^+}f_r(p,\omega_i,\omega_o)\cos\theta_id\omega_i
  $$
  因此环境光照可以转化为$\frac{\int_{\Omega_{f_r}}L_i(p,\omega_i)d\omega_i}{\int_{\Omega_{f_r}}d\omega_i}$和$\int_{\Omega^+}f_r(p,\omega_i,\omega_o)\cos\theta_id\omega_i$两项分别计算（Split Sum Approximation）。

* $\frac{\int_{\Omega_{f_r}}L_i(p,\omega_i)d\omega_i}{\int_{\Omega_{f_r}}d\omega_i}$的计算

  * 方法：在预处理阶段，对环境纹理进行Prefiltering。在对环境贴图进行Prefiltering时，要考虑粗糙度，因为随着粗糙度的增加，参与环境贴图Prefiltering的采样向量会更加分散，导致反射更为模糊。所以可以对于Prefiltering的每个粗糙度级别，按顺序将Prefiltering后的结果存储在Prefiltering贴图的Mipmap中。

    <img src="D:\Code\CG\202\GAMES202-note\Environment_mapping.assets\image-20210413181433913.png" alt="image-20210413181433913" style="zoom:50%;" />

  * 原理：在计算环境光照时，根据BRDF lobe的大小，在BRDF lobe的范围内，对环境光贴图进行采样，计算加权平均后的结果，相当于在镜面反射的方向上，先对环境光贴图上大小相当的区域做加权平均，然后取得平均值计算BRDF。对于漫反射，可以认为其BRDF lobe的中心方向为法线的方向。

    <img src="D:\Code\CG\202\GAMES202-note\Environment_mapping.assets\image-20210413181612024.png" alt="image-20210413181612024" style="zoom:50%;" />

  * 计算（以[Real Shading in Unreal Engine 4](https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf)为例）：

    
  
* $\int_{\Omega^+}f_r(p,\omega_i,\omega_o)\cos\theta_id\omega_i$的计算

  

  

