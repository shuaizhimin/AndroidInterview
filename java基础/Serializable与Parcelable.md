* 序列化:把对象转换为字节序列并存储在一个存储媒介的过程
* 反序列化:把字节序列化恢复为java对象
transient 不序列化

##序列化的目的
* 永久保存对象数据
* 通过序列化操作将对象数据在网络上进行传输(由于网络传输是以字节流的方式对数据进行传输的.因此序列化的目的是将对象数据转换成字节流的形式)
* 将对象数据在进程之间进行传递(Activity之间传递对象数据时,需要在当前的Activity中对对象数据进行序列化操作.在另一个Activity中需要进行反序列化操作讲数据取出)
* Java平台允许我们在内存中创建可复用的Java对象，但一般情况下，只有当JVM处于运行时，这些对象才可能存在，即，这些对象的生命周期不会比JVM的生命周期更长（即每个对象都在JVM中）但在现实应用中，就可能要停止JVM运行，但有要保存某些指定的对象，并在将来重新读取被保存的对象。这是Java对象序列化就能够实现该功能。（可选择入数据库、或文件的形式保存）
* 序列化对象的时候只是针对变量进行序列化,不针对方法进行序列化.
* 在Intent之间,基本的数据类型直接进行相关传递即可,但是一旦数据类型比较复杂的时候,就需要进行序列化操作了.

 
###Serializable、Parcelable区别 

* 在内存的使用中,Parcelable在性能好

> Serializable在序列化操作的时候会产生大量的临时变量,(原因是使用了反射机制)从而导致GC的频繁调用,因此在性能上会稍微逊色
> Parcelable是以Ibinder作为信息载体的.在内存上的开销比较小,因此在内存之间进行数据传递的时候,Android推荐使用Parcelable,既然是内存方面比价有优势,那么自然就要优先选择.

* 在读写数据的时候,Parcelable是在内存中直接进行读写,而Serializable是通过使用IO流的形式将数据读写入在硬盘上.

> 但是：虽然Parcelable的性能要强于Serializable,但是仍然有特殊的情况需要使用Serializable,而不去使用Parcelable,因为Parcelable无法将数据进行持久化,因此在将数据保存在磁盘的时候,仍然需要使用Serializable,因为Parcelable无法很好的将数据进行持久化.(原因是在不同的Android版本当中,Parcelable可能会不同,因此数据的持久化方面仍然是使用Serializable)


```
public class User implements Parcelable{
    public String userId;
    public int age;
    public Bitmap mBitmap;


    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(this.userId);
        dest.writeInt(this.age);
        dest.writeParcelable(this.mBitmap, 0);
    }

    public User() {
    }

    protected User(Parcel in) {
    //如果元素数据是list类型的时候需要： lits = new ArrayList<?> in.readList(list); 
    //否则会出现空指针异常.并且读出和写入的数据类型必须相同.如果不想对部分关键字进行序列化,可以使用transient关键字来修饰以及static修饰.
        this.userId = in.readString();
        this.age = in.readInt();
        this.mBitmap = in.readParcelable(Bitmap.class.getClassLoader());
    }

    public static final Creator<User> CREATOR = new Creator<User>() {
        public User createFromParcel(Parcel source) {
            return new User(source);
        }

        public User[] newArray(int size) {
            return new User[size];
        }
    };
```

#Parcelable源码
```
static jint android_os_Parcel_readInt(JNIEnv* env, jobject clazz){
 Parcel* parcel = parcelForJavaObject(env, clazz);
 if (parcel != NULL) {
  return parcel->readInt32();
 }
 return 0;
}


status_t Parcel::writeInt32(int32_t val){
 return writeAligned(val);
} 
template<class t="">
status_t Parcel::writeAligned(T val) {
 COMPILE_TIME_ASSERT_FUNCTION_SCOPE(PAD_SIZE(sizeof(T)) == sizeof(T));
  
 if ((mDataPos+sizeof(val)) <= mDataCapacity) {
restart_write:
  *reinterpret_cast<t*>(mData+mDataPos) = val;
  return finishWrite(sizeof(val));
 }
  
 status_t err = growData(sizeof(val));
 if (err == NO_ERROR) goto restart_write;
 return err;
}
真正的读写过程是由下面的源代码来完成的.
status_t Parcel::continueWrite(size_t desired)
{
 // If shrinking, first adjust for any objects that appear
 // after the new data size.
 size_t objectsSize = mObjectsSize;
 if (desired < mDataSize) {
  if (desired == 0) {
   objectsSize = 0;
  } else {
   while (objectsSize > 0) {
    if (mObjects[objectsSize-1] < desired)
     break;
    objectsSize--;
   }
  }
 }
  
 if (mOwner) {
  // If the size is going to zero, just release the owner's data.
  if (desired == 0) {
   freeData();
   return NO_ERROR;
  }
 
  // If there is a different owner, we need to take
  // posession.
  uint8_t* data = (uint8_t*)malloc(desired);
  if (!data) {
   mError = NO_MEMORY;
   return NO_MEMORY;
  }
  size_t* objects = NULL;
   
  if (objectsSize) {
   objects = (size_t*)malloc(objectsSize*sizeof(size_t));
   if (!objects) {
    mError = NO_MEMORY;
    return NO_MEMORY;
   }
 
   // Little hack to only acquire references on objects
   // we will be keeping.
   size_t oldObjectsSize = mObjectsSize;
   mObjectsSize = objectsSize;
   acquireObjects();
   mObjectsSize = oldObjectsSize;
  }
   
  if (mData) {
   memcpy(data, mData, mDataSize < desired ? mDataSize : desired);
  }
  if (objects && mObjects) {
   memcpy(objects, mObjects, objectsSize*sizeof(size_t));
  }
  //ALOGI("Freeing data ref of %p (pid=%d)\n", this, getpid());
  mOwner(this, mData, mDataSize, mObjects, mObjectsSize, mOwnerCookie);
  mOwner = NULL;
 
  mData = data;
  mObjects = objects;
  mDataSize = (mDataSize < desired) ? mDataSize : desired;
  ALOGV("continueWrite Setting data size of %p to %d\n", this, mDataSize);
  mDataCapacity = desired;
  mObjectsSize = mObjectsCapacity = objectsSize;
  mNextObjectHint = 0;
 
 } else if (mData) {
  if (objectsSize < mObjectsSize) {
   // Need to release refs on any objects we are dropping.
   const sp<ProcessState> proc(ProcessState::self());
   for (size_t i=objectsSize; i<mObjectsSize; i++) {
    const flat_binder_object* flat
     = reinterpret_cast<flat_binder_object*>(mData+mObjects[i]);
    if (flat->type == BINDER_TYPE_FD) {
     // will need to rescan because we may have lopped off the only FDs
     mFdsKnown = false;
    }
    release_object(proc, *flat, this);
   }
   size_t* objects =
    (size_t*)realloc(mObjects, objectsSize*sizeof(size_t));
   if (objects) {
    mObjects = objects;
   }
   mObjectsSize = objectsSize;
   mNextObjectHint = 0;
  }
 
  // We own the data, so we can just do a realloc().
  if (desired > mDataCapacity) {
   uint8_t* data = (uint8_t*)realloc(mData, desired);
   if (data) {
    mData = data;
    mDataCapacity = desired;
   } else if (desired > mDataCapacity) {
    mError = NO_MEMORY;
    return NO_MEMORY;
   }
  } else {
   if (mDataSize > desired) {
    mDataSize = desired;
    ALOGV("continueWrite Setting data size of %p to %d\n", this, mDataSize);
   }
   if (mDataPos > desired) {
    mDataPos = desired;
    ALOGV("continueWrite Setting data pos of %p to %d\n", this, mDataPos);
   }
  }
   
 } else {
  // This is the first data. Easy!
  uint8_t* data = (uint8_t*)malloc(desired);
  if (!data) {
   mError = NO_MEMORY;
   return NO_MEMORY;
  }
   
  if(!(mDataCapacity == 0 && mObjects == NULL
    && mObjectsCapacity == 0)) {
   ALOGE("continueWrite: %d/%p/%d/%d", mDataCapacity, mObjects, mObjectsCapacity, desired);
  }
   
  mData = data;
  mDataSize = mDataPos = 0;
  ALOGV("continueWrite Setting data size of %p to %d\n", this, mDataSize);
  ALOGV("continueWrite Setting data pos of %p to %d\n", this, mDataPos);
  mDataCapacity = desired;
 }
 
 return NO_ERROR;
}

```
* 整个读写全是在内存中进行，主要是通过malloc()、realloc()、memcpy()等内存操作进行，所以效率比JAVA序列化中使用外部存储器会高很多
* 读写时是4字节对齐的，可以看到#define PAD_SIZE(s) (((s)+3)&~3)这句宏定义就是在做这件事情
* 如果预分配的空间不够时newSize = ((mDataSize+len)*3)/2;会一次多分配50%
* 对于普通数据，使用的是mData内存地址，对于IBinder类型的数据以及FileDescriptor使用的是mObjects内存地址。后者是通过flatten_binder()和unflatten_binder()实现的，目的是反序列化时读出的对象就是原对象而不用重新new一个新对象


##总结
Java应用程序中有Serializable来实现序列化操作,Android中有Parcelable来实现序列化操作,相关的性能也作出了比较,因此在Android中除了对数据持久化的时候需要使用到Serializable来实现序列化操作,其他的时候我们仍然需要使用Parcelable来实现序列化操作,因为在Android中效率并不是最重要的,而是内存,通过比较Parcelable在效率和内存上都要优秀与Serializable,尽管Parcelable实现起来比较复杂,但是如果我们想要成为一名优秀的Android软件工程师,那么我们就需要勤快一些去实现Parcelable,而不是偷懒与实现Serializable.当然实现后者也不是不行,关键在于我们头脑中的那一份思想。