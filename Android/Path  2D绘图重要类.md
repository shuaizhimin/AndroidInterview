* moveTo 移动下一次操作的起点位置  不影响之前的操作/影响之后的操作
* setLastPoint  重置当前path中最后一个点位置，如果在绘制之前调用，效果和moveTo相同  影响之前的操作/影响之后的操作
* lineTo  添加上一个点到当前点之间的直线到Path
* close  连接第一个点连接到最后一个点，形成一个闭合区域
* ddRect, addRoundRect, addOval, addCircle, addPath, addArc, arcTo	添加(矩形， 圆角矩形， 椭圆， 圆， 路径， 圆弧) 到当前Path (注意addArc和arcTo的区别)
* offset	对当前路径之前的操作进行偏移(不会影响之后的操作)
* quadTo, cubicTo	分别为二次和三次贝塞尔曲线的方法
* rMoveTo, rLineTo, rQuadTo, rCubicTo	不带r的方法是基于原点的坐标系(偏移量)， rXxx方法是基于当前点坐标系(偏移量)
* setFillType, getFillType, isInverseFillType, toggleInverseFillType	设置,获取,判断和切换填充模式
* op	对两个Path进行布尔运算(即取交集、并集等操作)
* computeBounds	计算Path的边界
* reset, rewind	清除Path中的内容
reset不保留内部数据结构，但会保留FillType.
rewind会保留内部的数据结构，但不保留FillType
* transform	矩阵变换

> 弧度计算公式 2*Math.PI/360