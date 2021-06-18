## 在 Object 类中，不考虑重写的情况
### 内存地址
* 对象在内存中的位置，每个对象都是不同的
* 在java中内存中的对象地址是可变的（例如在内存回收的过程中内存地址就有可能发生变化）
### hashcode
内存地址通过哈希计算得出的一个数值，不同对象有可能相同
### equals
Object 内部是通过 == 实现，也就是说比较的是内存地址

## 如何获取到对象的内存地址 
```java
    public static Method getArrayBaseOffsetMethod() {
        try {
            Method method = Class.forName("sun.misc.Unsafe").getDeclaredMethod("arrayBaseOffset", Class.class);

            method.setAccessible(true);
            return method;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public static Method getAddressSizeMethod() {
        try {
            Method method = Class.forName("sun.misc.Unsafe").getDeclaredMethod("addressSize");

            method.setAccessible(true);
            return method;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public static Method getIntMethod() {
        try {
            Method method = Class.forName("sun.misc.Unsafe").getDeclaredMethod("getInt", Object.class, long.class);

            method.setAccessible(true);
            return method;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public static Method getLongMethod() {
        try {
            Method method = Class.forName("sun.misc.Unsafe").getDeclaredMethod("getLong", Object.class, long.class);

            method.setAccessible(true);
            return method;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public static long addressOf(Object o) throws Exception {
        
        Field field = Class.forName("sun.misc.Unsafe").getDeclaredField("theUnsafe");

        field.setAccessible(true);

        Object unsafeObject = field.get(null);

        Object[] array = new Object[]{o};

        int baseOffset = (int) getArrayBaseOffsetMethod().invoke(unsafeObject, Object[].class);

        int addressSize = (int) getAddressSizeMethod().invoke(unsafeObject);

        long objectAddress;

        switch (addressSize) {
            case 4:

                objectAddress = (int) getIntMethod().invoke(unsafeObject, array, baseOffset);

                break;

            case 8:

                objectAddress = (long) getLongMethod().invoke(unsafeObject, array, baseOffset);

                break;

            default:

                throw new Error("unsupported address size: " + addressSize);

        }
        return (objectAddress);

    }
```
