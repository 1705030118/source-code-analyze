- 单例
> 确保一个类只有一个实例
```java
public class Runtime {
    private static Runtime currentRuntime = new Runtime();

    /**
     * Returns the runtime object associated with the current Java application.
     * Most of the methods of class <code>Runtime</code> are instance
     * methods and must be invoked with respect to the current runtime object.
     *
     * @return  the <code>Runtime</code> object associated with the current
     *          Java application.
     */
    public static Runtime getRuntime() {
        return currentRuntime;
    }
}    
```
- 工厂方法
> 工厂方法把实例化操作推迟到子类
```java
public interface Collection<E> extends Iterable<E> {
    Iterator<E> iterator();
}
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public Iterator<E> iterator() {
                return new Itr();
    }
     /**
      * An optimized version of AbstractList.Itr
      */
     private class Itr implements Iterator<E> {
         int cursor;       // index of next element to return
         int lastRet = -1; // index of last element returned; -1 if no such
         int expectedModCount = modCount;
 
         Itr() {}
     }
}        
```
- 适配器
> 把一个类接口转换成另一个用户需要的接口
```java
public class Arrays {
    public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }
}
```
- 装饰者
> 为对象动态添加功能
```java
public class FilterInputStream extends InputStream {
    protected volatile InputStream in;
    protected FilterInputStream(InputStream in) {
        this.in = in;
    }
}
public class BufferedInputStream extends FilterInputStream {
    public BufferedInputStream(InputStream in) {
        this(in, DEFAULT_BUFFER_SIZE);
    }
    public BufferedInputStream(InputStream in, int size) {
        super(in);
        if (size <= 0) {
            throw new IllegalArgumentException("Buffer size <= 0");
        }
        buf = new byte[size];
    }
}    
```
- 代理
> 控制对其它对象的访问
```java
public class Proxy implements java.io.Serializable {
    public static Object newProxyInstance(ClassLoader loader,
                                              Class<?>[] interfaces,
                                              InvocationHandler h)
            throws IllegalArgumentException
        {
            Objects.requireNonNull(h);
    
            final Class<?>[] intfs = interfaces.clone();
            final SecurityManager sm = System.getSecurityManager();
            if (sm != null) {
                checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
            }
    
            /*
             * Look up or generate the designated proxy class.
             */
            Class<?> cl = getProxyClass0(loader, intfs);
    
            /*
             * Invoke its constructor with the designated invocation handler.
             */
            try {
                if (sm != null) {
                    checkNewProxyPermission(Reflection.getCallerClass(), cl);
                }
    
                final Constructor<?> cons = cl.getConstructor(constructorParams);
                final InvocationHandler ih = h;
                if (!Modifier.isPublic(cl.getModifiers())) {
                    AccessController.doPrivileged(new PrivilegedAction<Void>() {
                        public Void run() {
                            cons.setAccessible(true);
                            return null;
                        }
                    });
                }
                return cons.newInstance(new Object[]{h});
            } catch (IllegalAccessException|InstantiationException e) {
                throw new InternalError(e.toString(), e);
            } catch (InvocationTargetException e) {
                Throwable t = e.getCause();
                if (t instanceof RuntimeException) {
                    throw (RuntimeException) t;
                } else {
                    throw new InternalError(t.toString(), t);
                }
            } catch (NoSuchMethodException e) {
                throw new InternalError(e.toString(), e);
            }
        }
}
```