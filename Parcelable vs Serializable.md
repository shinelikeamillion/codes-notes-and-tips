## Parcelable vs Serializable
Serializable, the master of simplicity/极简之王
  * The beauty of serializable is that you only need to implement the Serializable interface on a class and its children. It is a marker interface, meaning that there is no method to implement, Java will simply do its best effort to serialize it efficiently.
  * The problem with this approach is that reflection/反射 is used and it is a slow process. This mechanism/机制 also tends to create a lot of temporary/临时 objects and cause quite a bit of garbage collection/垃圾回收.

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
Paracelable, the Speed King/效率之王
  * According to google engineers/工程师, this code will run significantly/显著地 faster. One of the reasons for this is that we are being explicit/明确的 about the serialization process/过程 instead of using reflection/反射 to infer it. It also stands to reason that the code has been heavily/严重的 optimized/优化 for this purpose.
  * However, it is obvious here that implementing Parcelable is not free. There is a significant/重要 amount/量 of boilerplate/样板 code and it makes the classes harder to read and maintain/维护.  
```
// access modifiers, accessors and regular constructors ommited for brevity
class ParcelableDeveloper implements Parcelable {
    String name;
    int yearsOfExperience;
    List<Skill> skillSet;
    float favoriteFloat;

    ParcelableDeveloper(Parcel in) {
        this.name = in.readString();
        this.yearsOfExperience = in.readInt();
        this.skillSet = new ArrayList<Skill>();
        in.readTypedList(skillSet, Skill.CREATOR);
        this.favoriteFloat = in.readFloat();
    }

    void writeToParcel(Parcel dest, int flags) {
        dest.writeString(name);
        dest.writeInt(yearsOfExperience);
        dest.writeTypedList(skillSet);
        dest.writeFloat(favoriteFloat);
    }

    int describeContents() {
        return 0;
    }


    static final Parcelable.Creator<ParcelableDeveloper> CREATOR
            = new Parcelable.Creator<ParcelableDeveloper>() {

        ParcelableDeveloper createFromParcel(Parcel in) {
            return new ParcelableDeveloper(in);
        }

        ParcelableDeveloper[] newArray(int size) {
            return new ParcelableDeveloper[size];
        }
    };

    static class Skill implements Parcelable {
        String name;
        boolean programmingRelated;

        Skill(Parcel in) {
            this.name = in.readString();
            this.programmingRelated = (in.readInt() == 1);
        }

        @Override
        void writeToParcel(Parcel dest, int flags) {
            dest.writeString(name);
            dest.writeInt(programmingRelated ? 1 : 0);
        }

        static final Parcelable.Creator<Skill> CREATOR
            = new Parcelable.Creator<Skill>() {

            Skill createFromParcel(Parcel in) {
                return new Skill(in);
            }

            Skill[] newArray(int size) {
                return new Skill[size];
            }
        };

        @Override
        int describeContents() {
            return 0;
        }
    }
}
```
