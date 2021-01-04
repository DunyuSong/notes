# String、StringBuffer、StringBuilder
## String 
String是不可变的对象，因此每次在对String类进行改变的时候都会生成一个新的string对象
## StringBuffer
StringBuffer是对象本身进行操作，而不是生成新的对象并改变对象引用

## StringBuilder
StringBuffer 兼容的 API，但不保证同步

## 使用策略
1、基本原则：如果要操作少量的数据，用String ；单线程操作大量数据，用StringBuilder ；多线程操作大量数据，用StringBuffer。
2、不要使用String类的”+”来进行频繁的拼接，因为那样的性能极差的，应该使用StringBuffer或StringBuilder类，这在Java的优化上是一条比较重要的原则。
3、 StringBuilder一般使用在方法内部来完成类似”+”功能，因为是线程不安全的，所以用完以后可以丢弃。StringBuffer主要用在全局变量中
4、相同情况下使用 StirngBuilder 相比使用 StringBuffer 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

