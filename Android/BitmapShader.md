### TileMode
 * CLAMP 复制边缘色彩 
 * REPEAT X、Y 轴进行重复图片显示，也就是平铺
 * MIRROR 在水平和垂直方向上使用交替镜像的方式重复图片的绘制



###绘制圆形bitmap
 ```
   BitmapShader  mBitmapShader=new BitmapShader(mBitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);
   //放置矩阵  
   mBitmapShader.setLocalMatrix(mMatrix);
   
   mBitmapPaint.setShader(mBitmapShader);
   //绘制圆形
   canvas.drawCircle(mDrawableRect.centerX(), mDrawableRect.centerY(), mDrawableRadius, mBitmapPaint);
 ```