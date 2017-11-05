###A->B         continue back
1. A:onCreate()->onStart()->onResume()
2. 跳转B: 
    A:onPause()
    B:onCreate()->onStart()->onResume()
    A:onSaveInstanceState()->onStop()
3. 返回:
    B:onPause()
    A:onRestart()->onStart()->onResume()
    B:onStop()->onDestroy()
4. Home键:
    A:onPause()->onStop()
   再次点击启动:
    A:onRestart()->onStart()->onResume()


    