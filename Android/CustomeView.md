* onMeasure   
 重写
```
@Override  
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
        // TODO Auto-generated method stub  
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);  
        int width = measureDimension(200, widthMeasureSpec);  
        int height = measureDimension(200, heightMeasureSpec);  
        setMeasuredDimension(width, height);  
    }  
  
    public int measureDimension(int defaultSize, int measureSpec){  
        int result;  
          
        int specMode = MeasureSpec.getMode(measureSpec);  
        int specSize = MeasureSpec.getSize(measureSpec);  
          
        if(specMode == MeasureSpec.EXACTLY){  
            result = specSize;  
        }else{  
            result = defaultSize;   //UNSPECIFIED  
            if(specMode == MeasureSpec.AT_MOST){  
                result = Math.min(result, specSize);  
            }  
        }  
        return result;  
    }  
```





### MeasureSpec  
* UNSPECIFIED 父容器不对View有任何限制，要多大给多大，这种情况一般用于系统内部，表示一种测量的状   态。
* EXACTLY 父容器已经测量出View所需要的精确大小，这个时候View的最终大小就是SpecSize所指定的值。它   对应于LayoutParams中的match_parent和具体的数值这两种模式。
* AT_MOST 父容器指定了一个可用大小即SpecSize，View大小不能大于这个值，具体是什么值要看不同View的   具体实现。它对应于LayoutParams中的Wrap_content