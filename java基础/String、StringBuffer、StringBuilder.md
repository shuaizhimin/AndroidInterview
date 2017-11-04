* 可变与不可变
  1.String final字符串 不可变 public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence
  2.StringBuffer、StringBuilder 可变
* 线程安全
   1.String 线程安全
   2.StringBuffer 线程安全
   3.StringBuilder 非线程安全

String equals 重写了方法 所以可以比较