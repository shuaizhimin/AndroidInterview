###add A Fragement 之后
onAttach()->onCreate()->onCreateView()->onViewCreated()->onActivityCreated()->onStart()->onResume()
### add A->add B
A:onAttach()->onCreate()->onCreateView()->onViewCreated()->onActivityCreated()->onStart()->onResume()
B:onAttach()->onCreate()->onCreateView()->onViewCreated()->onActivityCreated()->onStart()->onResume()
### add A-> replaceFragment B
 B: onAttach()->onCreate()
 A: onPause()->onStop()->onDestroyView()->onDestroy()->onDetach()
 B: onCreateView()->onViewCreated()->onActivityCreated()->onStart()->onResume()
### hide B

 