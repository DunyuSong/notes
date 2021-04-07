# Map 操作
## 遍历
### 方法一：键值都需要时使用
```java
    Map<Integer, Integer> map = new HashMap<Integer, Integer>();
    for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
        System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());
    }
```
### 方法二： 在for-each循环中遍历keys或values。（推荐）
```java
    Map<Integer, Integer> map = new HashMap<Integer, Integer>();
    //遍历map中的键
    for (Integer key : map.keySet()) {
        System.out.println("Key = " + key);
    }
    //遍历map中的值
    for (Integer value : map.values()) {
        System.out.println("Value = " + value);
    }
```
该方法比entrySet遍历在性能上稍好（快了10%），而且代码更加干净。
