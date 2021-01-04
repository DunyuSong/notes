### 枚举的定义
```java
public enum ItemType {
    ARTICLE("article"),     //文章
    PICSET("picset"),       //图集
    VIDEO("video"),         //视频
    USER("user"),           //用户
    GROUP("group"),         //群组
    OFFICIAL("official"),   //公众号
    PRODUCT("product"),     //商品
    MOMENT("moment")        //动态
    ;
}
```
### 获取枚举的值
```java

    private String objectType;

    ItemType(String objectType) {
        this.objectType = objectType;
    }

    public String getObjectType() {
        return objectType;
    }
```
### 判断给定的值是否在枚举中
```java
    //判断值是否在枚举中
    public static boolean contains(String value) {
        for (ItemType c : ItemType.values()) {
            if (c.name().equals(value)) {
                return true;
            }
        }
        return false;
    }
```
### 如果给定的值在枚举中则返回对饮枚举中的值
```java
    //如果给定的值在枚举中则返回对饮枚举中的值
    public static String isEnumValue(String value) {
        for (ItemType c : ItemType.values()) {
            if (c.name().equalsIgnoreCase(value)) {
                return c.name();
            }
        }
        return null;
    }
```
### 将字符串转换为枚举对应的枚举类型
- 有些时候我们定义方法的时候接受的是枚举类型，但是实际值是字符串类型，这时候我们需要将字符串类型传递给方法类型为枚举时可以这样做
- 例如一下方法接受的是枚举类型
```java
 public static String getOnesRecResult(String appName, String userId, SceneUser sceneUser, int itemNum, ItemType itemType, String objectId) {}
```
这是时候我们动态获取了一个值是"article",想将这个值传递给次方法，那么要进行一下转换：
```java
getOnesRecResult(appName, userId, SceneUser.RELATED_HOME, num, ItemType.valueOf(entityType.toUpperCase()), entityId);
```
> 参考：https://blog.csdn.net/xieyuooo/article/details/8483267

