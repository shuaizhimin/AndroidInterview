### Handler+Thead
  * 优缺点
    1. Handler用法简单明了，可以将多个异步任务更新UI的代码放在一起，清晰明了
    2. 处理单个异步任务代码略显多
  * 适用范围
    1. 多个异步任务的更新UI

### AsyncTask
  * 优缺点
    1. 处理单个异步任务简单，可以获取到异步任务的进度
    2. 可以通过cancel方法取消还没执行完的AsyncTask
    3. 处理多个异步任务代码显得较多
  * 适用范围
    1. 单个异步任务的处理

### ThreadPoolExecutor
  Executors提供了四种创建ExecutorService的方法，他们的使用场景如下：
   1. Executors.newCachedThreadPool()
    创建一个定长的线程池，每提交一个任务就创建一个线程，直到达到池的最大长度，这时线程池会保持长度不再变化
   2. Executors.newFixedThreadPool()
    创建一个可缓存的线程池，如果当前线程池的长度超过了处理的需要时，它可以灵活的回收空闲的线程，当需要增加时，
    它可以灵活的添加新的线程，而不会对池的长度作任何限制
   3. Executors.newScheduledThreadPool()
    创建一个定长的线程池，而且支持定时的以及周期性的任务执行，类似于Timer
  4. Executors.newSingleThreadExecutor()
    创建一个单线程化的executor，它只创建唯一的worker线程来执行任务

  * 适用范围
    1. 批处理任务

### IntentService
   1. 一个可以处理异步任务的简单Service