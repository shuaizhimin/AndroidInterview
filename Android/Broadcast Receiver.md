##问题
* 广播种类 
* 广播使用场景
* 广播跨进程通讯原理
* 

## 使用场景
 * App全局监听
 * 组件局部监听
 
>系统事件监听,支持进程间的通讯

## 广播的种类

###普通广播

通过Context.sendBroadcast()方法来发送，它是完全异步的。
这种方式效率更高，但是BroadcastReceiver无法使用setResult系列、getResult系列及abort（中止）系列API

###有序广播

是通过Context.sendOrderedBroadcast来发送，所有的receiver依次执行

> BroadcastReceiver可以使用setResult系列函数来结果传给下一个BroadcastReceiver，通过getResult系列函数来取得上个BroadcastReceiver返回的结果，并可以abort系列函数来让系统丢弃该广播，使用该广播不再传送到别的BroadcastReceiver。
可以通过在intent-filter中设置android:priority属性来设置receiver的优先级，优先级相同的receiver其执行顺序不确定。
如果BroadcastReceiver是代码中注册的话，且其intent-filter拥有相同android:priority属性的话，先注册的将先收到广播。
有序广播，即从优先级别最高的广播接收器开始接收，接收完了如果没有丢弃，就下传给下一个次高优先级别的广播接收器进行处理，依次类推，直到最后。如果多个应用程序设置的优先级别相同，则谁先注册的广播，谁就可以优先接收到广播。
这里接收短信的广播是有序广播，因此可以设置你自己的广播接收器的级别高于系统原来的级别，就可以拦截短信，并且不存收件箱，也不会有来信提示音。

### 本地广播
LocalBroadcastManager.getInstance(this).registerReceiver(br,new IntentFilter());
LocalBroadcastManager.getInstance(this).sendBroadcast(new Intent());
LocalBroadcastManager.getInstance(this).unregisterReceiver(br);
* 本地广播只能在自身App内传播，不必担心泄漏隐私数据
* 本地广播不允许其他App对你的App发送该广播，不必担心安全漏洞被利用
* 本地广播比全局广播更高效
* 以上三点都是源于其内部是用Handler实现的

### Sticky广播
sticky广播通过Context.sendStickyBroadcast()函数来发送

> 使用此函数需要发送广播时，需要获得BROADCAST_STICKY权
sendStickyBroadcast只保留最后一条广播，并且一直保留下去，这样即使已经有广播接收器处理了该广播，当再有匹配的广播接收器被注册时，此广播仍会被接收。如果你只想处理一遍该广播，可以通过removeStickyBroadcast()函数来实现。这里创建广播的过程和普通广播是一样的过程

###系统广播



## 广播源码

ContextImpl

```
public void sendBroadcast(Intent intent) {
        warnIfCallingFromSystemProcess();
        String resolvedType = intent.resolveTypeIfNeeded(getContentResolver());
        try {
            intent.prepareToLeaveProcess(this);
            ActivityManagerNative.getDefault().broadcastIntent(
                    mMainThread.getApplicationThread(), intent, resolvedType, null,
                    Activity.RESULT_OK, null, null, null, AppOpsManager.OP_NONE, null, false, false,
                    getUserId());
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
    }
```
ActivityManagerService
```
public final int broadcastIntent(IApplicationThread caller,
            Intent intent, String resolvedType, IIntentReceiver resultTo,
            int resultCode, String resultData, Bundle resultExtras,
            String[] requiredPermissions, int appOp, Bundle bOptions,
            boolean serialized, boolean sticky, int userId) {
        enforceNotIsolatedCaller("broadcastIntent");
        synchronized(this) {
            intent = verifyBroadcastLocked(intent);
            final ProcessRecord callerApp = getRecordForAppLocked(caller);
            final int callingPid = Binder.getCallingPid();
            final int callingUid = Binder.getCallingUid();
            final long origId = Binder.clearCallingIdentity();
            int res = broadcastIntentLocked(callerApp,
                    callerApp != null ? callerApp.info.packageName : null,
                    intent, resolvedType, resultTo, resultCode, resultData, resultExtras,
                    requiredPermissions, appOp, bOptions, serialized, sticky,
                    callingPid, callingUid, userId);
            Binder.restoreCallingIdentity(origId);
            return res;
        }
    }
    
    final int broadcastIntentLocked(ProcessRecord callerApp,
            String callerPackage, Intent intent, String resolvedType,
            IIntentReceiver resultTo, int resultCode, String resultData,
            Bundle resultExtras, String[] requiredPermissions, int appOp, Bundle bOptions,
            boolean ordered, boolean sticky, int callingPid, int callingUid, int userId) {
            ...
            ...
            ...
            //如果不是有序广播 加入广播队列
            if (!ordered && NR > 0) {
            // If we are not serializing this broadcast, then send the
            // registered receivers separately so they don't wait for the
            // components to be launched.
            if (isCallerSystem) {
                checkBroadcastFromSystem(intent, callerApp, callerPackage, callingUid,
                        isProtectedBroadcast, registeredReceivers);
            }
            final BroadcastQueue queue = broadcastQueueForIntent(intent);
            BroadcastRecord r = new BroadcastRecord(queue, intent, callerApp,
                    callerPackage, callingPid, callingUid, resolvedType, requiredPermissions,
                    appOp, brOptions, registeredReceivers, resultTo, resultCode, resultData,
                    resultExtras, ordered, sticky, false, userId);
            if (DEBUG_BROADCAST) Slog.v(TAG_BROADCAST, "Enqueueing parallel broadcast " + r);
            final boolean replaced = replacePending && queue.replaceParallelBroadcastLocked(r);
            if (!replaced) {
                queue.enqueueParallelBroadcastLocked(r);
                queue.scheduleBroadcastsLocked();
            }
            registeredReceivers = null;
            NR = 0;
        }
        ...
        ...
        sendPackageBroadcastLocked(cmd,new String[] {ssp}, userId);
            
      }


 private final void sendPackageBroadcastLocked(int cmd, String[] packages, int userId) {
        for (int i = mLruProcesses.size() - 1 ; i >= 0 ; i--) {
            ProcessRecord r = mLruProcesses.get(i);
            if (r.thread != null && (userId == UserHandle.USER_ALL || r.userId == userId)) {
                try {
                //发送广播 IApplicationThread
                    r.thread.dispatchPackageBroadcast(cmd, packages);
                } catch (RemoteException ex) {
                }
            }
        }
    }
```
ApplicationThreadNative

```
//处理广播
private final IBinder mRemote;
 public void dispatchPackageBroadcast(int cmd, String[] packages) throws RemoteException {
        Parcel data = Parcel.obtain();
        data.writeInterfaceToken(IApplicationThread.descriptor);
        data.writeInt(cmd);
        data.writeStringArray(packages);
        mRemote.transact(DISPATCH_PACKAGE_BROADCAST_TRANSACTION, data, null,
                IBinder.FLAG_ONEWAY);
        data.recycle();
    }
    
```