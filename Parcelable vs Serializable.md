## Parcelable vs Serializable
Serializable： 数据持久化（保存对象属性到本地文件，数据库，网络流以方便数据传输）
  * The beauty of serializable is that you only need to implement the Serializable interface on a class and its children. It is a marker interface, meaning that there is no method to implement, Java will simply do its best effort to serialize it efficiently.
  * The problem with this approach is that reflection is used and it is a slow process. This mechanism also tends to create a lot of temporary objects and cause quite a bit of garbage collection.

```
// modifiers/修饰符 and constructors/构造函数 amitted/省略 for brevity/简短
public class SerializableDeveloper implements Serializable {
    String name;
    int yearsOfExperience;
    List<Skill> skillSet;
    float favoriteFloat;

    static class Skill implements Serializable {
        String name;
        boolean programmingRelated;
    }
}
```
