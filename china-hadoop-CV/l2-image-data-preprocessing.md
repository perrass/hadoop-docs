# Image Data Processing

### 图片存储原理

RGB颜色空间(b, g, r)，取值范围[0, 255]或者[0.0, 1.0]

灰度处理

Gray = R * 0.299 + G * 0.587 + B * 0.114

### 空域分析及变换

滤波/卷积：在每个图片位置（x,y）上进行基于邻域的函数计算

padding类型

* 补零（ze'ro-padding）
* 边界复制（replication）
* 镜像（reflection）
* 块复制（wraparound）

### 频域分析及变换

### 金字塔

### 模板匹配

### 代码实践