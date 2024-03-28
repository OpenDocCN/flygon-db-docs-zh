# 13.4.2 OpenGIS 几何模型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/opengis-geometry-model.html`](https://dev.mysql.com/doc/refman/8.0/en/opengis-geometry-model.html)

13.4.2.1 几何类层次结构

13.4.2.2 几何类

13.4.2.3 点类

13.4.2.4 曲线类

13.4.2.5 线串类

13.4.2.6 表面类

13.4.2.7 多边形类

13.4.2.8 几何集合类

13.4.2.9 多点类

13.4.2.10 多曲线类

13.4.2.11 多线串类

13.4.2.12 多表面类

13.4.2.13 多边形集合类

OGC 的**带有几何类型的 SQL**环境提出的几何类型集合基于**OpenGIS 几何模型**。在这个模型中，每个几何对象具有以下一般属性：

+   它与空间参考系统相关联，描述了对象定义的坐标空间。

+   它属于某些几何类。