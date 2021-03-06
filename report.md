# 计算机图形学 大作业报告

## 绘制算法简介

- 算法类型：光线追踪/路径追踪
  - 支持**漫反射、反射、折射**及其任意组合：在每个表面将会分别判断是否需要漫反射/反射/折射并且发射相应追踪光线。需要满足漫反射率、反射率和折射率的和为1。
  - 如果发生漫反射：对于光线追踪将会判断是否为阴影，若不为阴影则加入来自光源的照明。对于路径追踪将会进行两类采样，分别为在每一个光源上均匀采样一个出射方向，以及在半球面上均匀采样一个出射方向。这是为了减少光源过小带来的采样亮度方差大问题。
  - 如果发生反射/折射：出射方向是固定的。根据菲涅尔折射原理，随着入射角变化，折射率和反射率也会相对发生变化，入射角越大，光线越倾向于反射。代码中近似模拟了这一变化。
  - 关于光源：光线追踪支持点光源和方向光源。路径追踪支持面光源（矩形）。二者均支持环境光，默认环境光在所有方向等大。
  - 支持**三角网格**导入：通过obj文件读入三角网格。支持读入带有纹理/发现信息的三角网格。支持读入四角网格（将会被转化成三角网格）。

- 光学
  - **景深**：支持景深，通过模拟光线经过光圈内某一点然后打到光屏上。需要在设置相机时指定光圈大小（记录为焦距和光圈半径的比值），对焦的物距，以及需要在src/main.cpp中设定随机采样次数。
  - **抗锯齿**：支持抗锯齿，通过对每一个像素位置在一定的误差范围内重复采样光线得到。需要在src/main.cpp中指定采样方法和次数。
  - **软阴影**：路径追踪通过引入面光源以及面光源采样的方式自动实现软阴影效果。
    
- 复杂面片
  - **法向插值**：法向插值与否为可调参数，需要指定。如果obj文件中没有逐面片记录的法向信息且仍需要插值，则会将每个顶点连接的所有面片的法向均值作为顶点法向并基于此进行插值。
  - **贴图**：纹理贴图以图片形式读入，需要用到opencv库。支持读入颜色（漫反射）贴图，光滑度（镜面反射）贴图，**法向贴图**，发光贴图，环境光遮挡贴图。其中法向贴图默认使用切线空间。使用贴图的前提是obj文件中有逐面片的uv坐标信息。贴图和法向插值可以同时使用。
  - **Kd-tree加速**：使用Kd-tree加速面片求交。具体方式为，对每个三角网格计算一个包围盒，轮流在三个轴方向上找到坐标中位顶点，分割包围盒形成两个子网格，以此类推。求交时可以排除与不相交的包围盒内网格求交，因此可以加速。
  
- 参数曲面
  - 支持**旋转Bezier曲线、B样条曲面解析法求交**：对于给定的曲线（必须在XY平面内），默认绕Y轴旋转，首先生成包围盒判断与包围盒的交点，然后均匀采样光线与包围盒相交范围内若干点作为初始值，通过牛顿迭代的方式缩小误差，误差小于一定阈值则认为相交。相关参数需要在include/revsurface.hpp中设定。

## 得分点

- 光线追踪、路径追踪
- 景深、软阴影、抗锯齿、贴图（漫反射贴图、法向贴图等）
- 参数曲面解析法求交
- 复杂网格模型（Kd-tree/包围盒、法向插值）

## 代码片段

- 光线追踪：src/main.cpp: 274~393
- 路径追踪：src/main.cpp: 26~271 （random_sample函数为随机采样漫反射出射方向）
- 景深：include/camera.hpp: 70-103
  - 调节景深：txt场景文件中f_a光圈大小和f_u对焦物距 + src/main.cpp中repeat_aperture参数
- 软阴影/面光源： include/surfacelight.hpp
  - 调用：src/main.cpp: 134~149
- 抗锯齿：src/main.cpp中repeat_x, repeat_y, repeat参数
- 贴图：include/texture.hpp
  - 调用：include/mesh.hpp, src/mesh.cpp, include/meshtriangle.hpp
  - 定义：见txt场景文件
- 参数曲面解析法求交：
  - 参数曲面：include/curve.hpp
  - 求交：include/revsurface.hpp (solve函数为牛顿法)
- 复杂网格
  - Kd-tree: src/mesh.cpp中computeKdTree函数
  - 求交：src/mesh.cpp中intersect函数
  - 读取：src/mesh.cpp中构造函数
  - 三角面片/法向插值：include/meshtriangle.hpp


## 结果解释

- 运行run_all.sh可以自动编译。编辑run_all.sh以指定输入文件名（输入为txt场景文件），输出文件名，以及采用光线追踪/路径追踪。
- **scene01**：花园/植物。场景文件为testcases/scene01_path_tracing.txt。采用路径追踪。体现的功能包括：路径追踪、面光源/软阴影/环境光、反射/折射/透明效果、复杂网格/法向插值、漫反射贴图/环境光遮挡贴图/法向贴图、旋转曲面、景深、抗锯齿。
- **scene02**：科幻/太空飞船。场景文件为testcases/scene02_path_tracing.txt。采用路径追踪。体现的功能包括：复杂网格/法向插值、漫反射贴图/镜面反射贴图/环境光遮挡贴图/法向贴图/发光贴图。
- **scene03**：scene01的光线追踪版本。场景文件为testcases/scene03_ray_tracing.txt。额外的改动包括：采用点光源；更改了旋转曲面的形状和材质（scene01中为Bezier曲线，本图为B样条）；加入透明复杂面片。
- **demo**：一些朴素的场景，体现算法的一般效果，也可以用于复现，位于code/output/。其中demo.bmp对应的场景文件为testcases/demo.txt。


    