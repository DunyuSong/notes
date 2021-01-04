# TestNG DataProvider 数据提供者
- 数据提供者是用 @DataProvider标注的方法。这个annotation只有一个字符串属性：它的名称。如果没有提供名称，数据提供者的名称就默认采用方法的名称。
- 数据提供者返回一些java对象，这些对象将作为参数传递给一个@Test标注的方法。从哪个数据提供者接受参数是在@Test annotation的 DataProvider属性中指定的。
- 从本质上来说，数据提供者同时实现两个目的：
    1. 向测试方法传递任意数目的参数（可以是任何Java类型）
    2. 根据需要，允许利用不同的参数集合对它的测试方法进行多次调用。

### 针对数据提供者的参数
- 数据提供者本身可以接受两个类型的参数：Method 和 ITestContext。TestNG在调用数据提供者之前会设置这两个参数。所以以下方法方法的签名都是有效的,返回返回值必须是Object[][]、Object[]、Iterator<Object[]>。Object[][]和Object[]都是实现了implements Iterator<Object[]>接口的类，所以当然我们也可以自定义一个数据提供者，只要实现了Iterator接口即可。
```java
    @DataProvider(name = "dataProviderBean")
    public Object[][] providerNumbers(Method method, ITestContext iTestContext) {...}
    
    @DataProvider(name = "dataProviderBean")
    public Object[] providerNumbers(Method method) {...}
    
    @DataProvider(name = "dataProviderBean")
    public Iterator<Object[]> providerNumbers(ITestContext iTestContext) {...}
    
    @DataProvider(name = "dataProviderBean")
    public Object[][] providerNumbers() {...}
```
#### 如何实现自动给参数赋值，以及为什么DataProvider要求参数的返回类型必须是二维数组、一维数组、Iterator
- 源码解析 MethodInvocationHelper.java
```
  protected static Iterator<Object[]> invokeDataProvider(Object instance, Method dataProvider,
      ITestNGMethod method, ITestContext testContext, Object fedInstance,
      IAnnotationFinder annotationFinder) {
    
    // step1:获取获取所有参数，并且给参数初始化
    List<Object> parameters = getParameters(dataProvider, method, testContext, fedInstance, annotationFinder);
    
    Object result = invokeMethodNoCheckedException(dataProvider, instance, parameters);
    if (result == null) {
      throw new TestNGException("Data Provider " + dataProvider + " returned a null value");
    }
    
    // 检查方法的返回值类型是否是以下类型
    // If it returns an Object[][] or Object[], convert it to an Iterator<Object[]>
    if (result instanceof Object[][]) {
      return new ArrayIterator((Object[][]) result);
    } else if (result instanceof Object[]) {
      return new OneToTwoDimArrayIterator((Object[]) result);
    } else if (result instanceof Iterator) {
      Type returnType = dataProvider.getGenericReturnType();
      if (returnType instanceof ParameterizedType) {
        ParameterizedType contentType = (ParameterizedType) returnType;
        Class<?> type = (Class<?>) contentType.getActualTypeArguments()[0];
        if (type.isArray()) {
          return (Iterator<Object[]>) result;
        } else {
          return new OneToTwoDimIterator((Iterator<Object>) result);
        }
      } else {
        // Raw Iterator, we expect user provides the expected type
        return (Iterator<Object[]>) result;
      }
    }
    throw new TestNGException("Data Provider " + dataProvider + " must return"
          + " either Object[][] or Object[] or Iterator<Object[]> or Iterator<Object>, not " + dataProvider.getReturnType());
  }
```
- getParameters()方法源码
```java

  private static List<Object> getParameters(Method dataProvider, ITestNGMethod method, ITestContext testContext,
                                            Object fedInstance, IAnnotationFinder annotationFinder) {
    // Go through all the parameters declared on this Data Provider and
    // make sure we have at most one Method and one ITestContext.
    // Anything else is an error
    List<Object> parameters = new ArrayList<>();
    Collection<Pair<Integer, Class<?>>> unresolved = new ArrayList<>();
    ConstructorOrMethod com = method.getConstructorOrMethod();
    int i = 0;
    // 遍历所有参数类型，并且初始化
    for (Class<?> cls : dataProvider.getParameterTypes()) {
      if (cls.equals(Method.class)) {
        parameters.add(com.getMethod());
      } else if (cls.equals(Constructor.class)) {
        parameters.add(com.getConstructor());
      } else if (cls.equals(ConstructorOrMethod.class)) {
        parameters.add(com);
      } else if (cls.equals(ITestNGMethod.class)) {
        parameters.add(method);
      } else if (cls.equals(ITestContext.class)) {
        parameters.add(testContext);
      } else if (cls.equals(Class.class)) {
        parameters.add(com.getDeclaringClass());
      } else {
        boolean isTestInstance = annotationFinder.hasTestInstance(dataProvider, i);
        if (isTestInstance) {
          parameters.add(fedInstance);
        } else {
          unresolved.add(new Pair<Integer, Class<?>>(i, cls));
        }
      }
      i++;
    }
    if (!unresolved.isEmpty()) {
      StringBuilder sb = new StringBuilder();
      sb.append("Some DataProvider ").append(dataProvider).append(" parameters unresolved: ");
      for (Pair<Integer, Class<?>> pair : unresolved) {
        sb.append(" at ").append(pair.first()).append(" type ").append(pair.second()).append("\n");
      }
      throw new TestNGException(sb.toString());
    }
    return parameters;
  }
```




# 参考
- JAVA测试新技术：TESTNG和高级概念
- 