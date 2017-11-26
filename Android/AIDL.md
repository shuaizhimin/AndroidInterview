##创建.aidl文件
AIDL支持的数据类型
* Java中所有原语类型(int、long、char、boolean等)
* String
* CharSequence
* List
* Map


2.实现接口

> 注意: 
>  * 多线程保证线程安全
>  * 耗时操作准备
>  * 引发的任何异常都不会回传给调用方

3.向客户端公开该接口