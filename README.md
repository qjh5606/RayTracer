# RayTracer
An Ray Tracer which I learn the basic Ray Tracing Knowledge.

> https://github.com/ssloy/tinyraytracer/wiki/Part-1:-understandable-raytracing


#  一个简单光线追踪渲染器
简单的介绍可以参考:[图形学基础 | 光线追踪](https://blog.csdn.net/qjh5606/article/details/89202746)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190411222303563.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FqaDU2MDY=,size_16,color_FFFFFF,t_70)

单独看一个像素上的追踪过程的话:
1. 从视线方向发射一条射线;
2. 取得射线与场景最近的交点;
3. 取得交点的材质颜色;
4. 如果材质包含反射或折射,则改变光线的方向;
5. 寻找下一个交点.并重复(2). 直到在场景内找不到交点.或者达到达到最大跟踪次数.
6. 对于(2). 因为需要遍历场景. 开销随场景复杂度上升而上升.

对于一些简单的示例场景中, 可以简化为 射线和球. 或者射线与平面的相交. 求最近的交点.

> 以下是实现代码的几个问题

### 光线投射
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190411223447185.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FqaDU2MDY=,size_16,color_FFFFFF,t_70)
对像素空间的每一点,变换到世界坐标系中. 
发射一条光线,对光线进行跟踪. 
```
float aspect = width / (float)height; // 防止失真
	for (size_t j = 0; j < height; j++) {
		for (size_t i = 0; i < width; i++) {
			// 将像素位置转换成为真实世界坐标位置
			/*
				(i+0.5) 像素偏移量
				+(i+0.5)/(width/2.0)* tan(fov / 2.)*aspect 世界坐标系偏移量
				起始世界坐标 - 1*tan(fov / 2.)*aspect
			*/
			float x = (2* (i + 0.5) / (float)width - 1)*tan(fov / 2.)*aspect;
			float y = -(2 * (j + 0.5) / (float)height - 1)*tan(fov / 2.);
			// 发出的光线方向
			Vec3f dir = Vec3f(x, y, -1).normalize(); 
			framebuffer[i + j*width] = cast_ray(Vec3f(0, 0, 0), dir, spheres,lights);
		}
	}
```

### 射线与球的相交
参考之前的博客 [射线与球的相交](https://blog.csdn.net/qjh5606/article/details/86699867)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190411222445191.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FqaDU2MDY=,size_16,color_FFFFFF,t_70)
### 反射光向量求解
[几何向量：计算光线反射reflect向量](https://blog.csdn.net/qjh5606/article/details/88737470)

### 折射光向量refract的计算
[折射光向量refract的计算](https://blog.csdn.net/qjh5606/article/details/89220500)

### 如何产生阴影
就是求出射线与物体的交点之后.
求解出 交点与 光源i 之间的 射线 `light_dir` 
对 `light_dir` 进行跟踪. 
从交点出发,看看能不能找到与它相交的物体.
如果可以,那么就是处于 该光源i的阴影处. 不应该进行光照计算.

```
// 应用光照
	float diffuse_light_intensity = 0, specular_light_intensity = 0;
	for (size_t i = 0; i < lights.size(); i++) {
		// 光源的方向:交点指向光源
		Vec3f light_dir = (lights[i].position - point).normalize(); 
		float light_distance = light_dir.norm();  // 距离

		// 判断是否 light[i]这个光线找不到point
		Vec3f shdow_orig = light_dir*N<0 ? point - N*1e-3 : point + N*1e-3; 
		Vec3f shadow_pt, shadow_N;
		Material tempmaterial;
		// 就是说从这个点出发,看看能不能找到与它相交的sphere,如果可以,那么就是处于阴影.lights[i]找不到
		if (scene_intersect(shdow_orig, light_dir, spheres, shadow_pt, shadow_N, tempmaterial))
			continue;

		diffuse_light_intensity += lights[i].intensity * std::max(0.f, light_dir*N);
		specular_light_intensity += powf(std::max(0.f, -reflect(-light_dir, N)*dir), material.specular_exponent)
			*lights[i].intensity;
	}
```
### 

### 渲染效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190411223643940.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FqaDU2MDY=,size_16,color_FFFFFF,t_70)
 

 








