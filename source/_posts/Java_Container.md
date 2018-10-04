---
title: Container
date: 2018/8/5
categories: Java
---
## Basic Concept
容器是用来存储对象的，通常有两种
 - *Collection: 一个独立的序列，例如List，Set等*
 - *Map:一种键-值对.* 
## List、Set、Map的区别
Now check the different result on the demo.
```java
public class PrintingContainers {  
    public static void main(String[] args) {  
        System.out.println(fill(new ArrayList<>()));  
  System.out.println(fill(new LinkedList<>()));
  System.out.println(fill(new HashSet<>()));  
  System.out.println(fill(new TreeSet<>()));  
  System.out.println(fill(new LinkedHashSet<>()));  
  
  System.out.println(fill(new HashMap<>()));  
  System.out.println(fill(new TreeMap<>()));  
  System.out.println(fill(new LinkedHashMap<>()));  
  }  
  
    static Collection fill(Collection<String> collection) {  
        collection.add("dog");  
  collection.add("cat");  
  collection.add("rat");  
  collection.add("dog");  
 return collection;  
  }  
  
    static Map fill(Map<String, String> map) {  
        map.put("rat", "Tom");  
  map.put("cat", "Jerry");  
 return map;  
  }  
}
```
Result :
```
[dog, cat, rat, dog]
[dog, cat, rat, dog]
[rat, cat, dog]
[cat, dog, rat]
[dog, cat, rat]
{rat=Tom, cat=Jerry}
{cat=Jerry, rat=Tom}
{rat=Tom, cat=Jerry}
```
## List
 - ArrayList: 适合查询，不适合插入和删除元素。
 - LinkedList : 按照添加的顺序存储元素，常用于频繁的插入和删除元素。

*Practice For List: Create a single linked List.User iterator to insert or remove elements*
```java
public class SingleListDemo {  
    public static void main(String[] args) {  
        SList<String> list = new SList<>();  
  SListIterator iterator = list.iterator();  
  iterator.insert("wang");  
  System.out.println(list);  
  iterator.insert("shen");  
  System.out.println(list);  
  iterator.insert("xing");  
  System.out.println(list);  
  
  SListIterator iterator1 = list.iterator();  
  iterator1.remove();  
  System.out.println(list);  
  }  
}  
  
class SList<E> {  
    Link<E> head = new Link<>(null);  
  
 public SListIterator iterator() {  
        return new SListIterator<E>(head);  
  }  
  
    @Override  
  public String toString() {  
        if (head.next == null) {  
            return "[]";  
        } else {  
            System.out.print("[");  
            StringBuilder stringBuilder = new StringBuilder();  
            SListIterator iterator = this.iterator();  
            while (iterator.hasNext()) {  
                stringBuilder.append(iterator.next() + (iterator.hasNext() ? "," : ""));  
            }  
            return stringBuilder + "]";  
        }  
    }  
}  
  
class Link<E> {  
    E e;  
    Link<E> next;  
    public Link(E e) {  
        this.e = e;  
    }  
    public Link(E e, Link<E> next) {  
        this.e = e;  
        this.next = next;  
    }  
    @Override  
    public String toString() {  
        return e != null ? e.toString() : null;  
    }  
}  
  
class SListIterator<E> {  
    Link<E> current;  
  
    public SListIterator(Link<E> current) {  
        this.current = current;  
    }  
  
    public boolean hasNext() {  
        return current.next != null;  
    }  
  
    public void insert(E e) {  
        current.next = new Link<>(e);  
        current = current.next;  
    }  
  
    public void remove() {  
        if (current.next != null)  
            current.next = current.next.next;  
    }  
  
    public Link<E> next() {  
        current = current.next;  
        return current;  
    }  
}
```
## Set
 - HashSet: 获取元素最快的一种
 - TreeSet: 降序的方式存储元素
 - LinkedHashSet : 根据添加的顺序存储元素
 - SortedSet: 由TreeSet实现的接口

*Practice:Create a SortedSet ,Using LinkerList as an underlying implementation*
```java
public class MySortedSetDemo {  
    public static void main(String[] args) {  
        MySortedSet<Integer> integerMySortedSet = new MySortedSet<>();  
        integerMySortedSet.add(6);  
        integerMySortedSet.add(8);  
        integerMySortedSet.add(2);  
        integerMySortedSet.add(5);  
        integerMySortedSet.add(2);  
        integerMySortedSet.add(7);  
        System.out.println(integerMySortedSet);  
        MySortedSet<String> stringMySortedSet = new MySortedSet<>();  
        stringMySortedSet.add("wang");  
        stringMySortedSet.add("shen");  
        stringMySortedSet.add("xing");  
        stringMySortedSet.add("is");  
        stringMySortedSet.add("good");  
        stringMySortedSet.add("is");  
        stringMySortedSet.add("cool");  
        System.out.println(stringMySortedSet);  
    }  
}  
class MySortedSet<E> extends LinkedList<E> {  
    public int compare(E e1, E e2) {  
        System.out.println(e1.hashCode());  
        System.out.println(e2.hashCode());  
        int result = e1.hashCode() - e2.hashCode();  
        return Integer.compare(result, 0);  
    }  
    public boolean add(E e) {  
        if (!this.contains(e)) {  
            Iterator<E> iterator = this.iterator();  
            int index = 0;  
            while (iterator.hasNext()) {  
                if (compare(iterator.next(), e) < 1) {  
                    index++;  
                }  
            }  
            add(index, e);  
            return true;  
        }  
        return false;  
  }  
}
```
## Map
 - HashMap: 最快的找到元素的方式
 - TreeMap:降序方式存储，且是唯一带有submap方法的Map，可以返回一个子树
 - LinkerHashMap: 按照添加的顺序存储



## Something about Array.asList
 ```java
List<Integer> list = Arrays.asList(1,2,3);  
list.add(4);
//Runtime error Because the array cannot be resized
 ```
Let's check the source code of Array.asList. It will return a fixed-size list by the specific array.
```java
public static <T> List<T> asList(T... a) {  
    return new ArrayList<>(a);  
}
```
## 散列与散列码
由于线性搜索的速度相当慢，因此在HashMap中采用了散列码来取代线性搜索。它是一个代表对象的int值。hashCode()属于Object中的方法，因此适用于所有的java对象。HashMap就是通过hashCode来进行查询的。下面是将Groundhog和Prediction联系起来的demo：
```java
public class Prediction {
    private static Random random = new Random(47);
    private boolean shadow = random.nextDouble() > 0.5;
    @Override
    public String toString() {
        if (shadow) {
            return "Six more ";
        } else {
            return "early spring";
        }
    }
}
public class Groundhog {
    protected int number;

    public Groundhog(int number) {
        this.number = number;
    }

    @Override
    public String toString() {
        return "Groundhog #" + number;
    }
}
public static <T extends Groundhog> void detectSpring(Class<T> type) throws Exception {
        Constructor<T> ghog = type.getConstructor(int.class);
        Map<Groundhog, Prediction> map = new HashMap<>();
        for (int i = 0; i < 10; i++) {
            map.put(ghog.newInstance(i), new Prediction());
        }
        System.out.println("map = " + map);
        Groundhog gh = ghog.newInstance(3);
        if (map.containsKey(gh)) {
            System.out.println(map.get(gh));
        }
    }
```
结果是找不到该key，原因很简单，这两个根本不是同一个类当然是查不到的，而它的查找方式就是在一个node中查找是否包含某个对象的hashcode，object的默认hashcode是和地址相关的。而想要让他们联系起来则需要覆写hashcode方法和equals方法。修改GroundDog
```java
public class Groundhog {
    protected int number;

    public Groundhog(int number) {
        this.number = number;
    }

    @Override
    public String toString() {
        return "Groundhog #" + number;
    }

    @Override
    public boolean equals(Object o) {
        return number == ((Groundhog) o).number;
    }

    @Override
    public int hashCode() {
        return number;
    }
}
```
那么为什么也要覆写equals方法呢，因为HashMap中覆写了equals方法，是根据key来判断是否相同的
```java
public final boolean equals(Object var1) {
            if (var1 == this) {
                return true;
            } else {
                if (var1 instanceof Entry) {
                    Entry var2 = (Entry)var1;
                    if (Objects.equals(this.key, var2.getKey()) && Objects.equals(this.value, var2.getValue())) {
                        return true;
                    }
                }

                return false;
            }
        }
```
创建一个新的Map：
```java
public class SlowMap<K, V> extends AbstractMap<K, V> {
    private List<K> keys = new ArrayList<>();
    private List<V> values = new ArrayList<>();

    public V put(K key, V value) {
        V oldValue = get(key);
        if (!keys.contains(key)) {
            keys.add(key);
            values.add(value);
        } else {
            values.set(keys.indexOf(value), value);
        }
        return oldValue;
    }

    public V get(Object key) {
        if (!keys.contains(key))
            return null;
        else
            return values.get(keys.indexOf(key));
    }

    @Override
    public Set<Map.Entry<K, V>> entrySet() {
        Set<Map.Entry<K, V>> set = new HashSet<>();
        Iterator<K> ki = keys.iterator();
        Iterator<V> vi = values.iterator();
        while (ki.hasNext()) {
            set.add(new MapEntry<K, V>(ki.next(), vi.next()));
        }
        return null;
    }
}

class MapEntry<K, V> implements Map.Entry<K, V> {
    private K key;
    private V value;

    public MapEntry(K key, V value) {
        this.key = key;
        this.value = value;
    }

    @Override
    public K getKey() {
        return key;
    }

    @Override
    public V getValue() {
        return value;
    }

    @Override
    public V setValue(V v) {
        V oldValue = value;
        value = v;
        return oldValue;
    }
}
```
这个简陋的map问题在于对键的保存是线性，只能挨个匹配，这样是非常慢的，那么HashMap是如何优化的呢，其实HashMap的数据结构是数组加单向链表的形式。将hashcode的hash值（就是一个转换的方法）作取模运算，结果作为数组的索引值index，而数组的元素是单项链表，挨个add。当我们需要get的时候则先通过hashcode的hash值作取模运算得到index。然后再在取得链表中挨个配对。简单实现：
```java
SimpleHashMap.java
public class SimpleHashMap<K, V> extends AbstractMap<K, V> {
    static final int SIZE = 997;
    LinkedList<MapEntry<K, V>>[] buckets = new LinkedList[SIZE];

    @Override
    public Set<Map.Entry<K, V>> entrySet() {
        Set<Map.Entry<K, V>> set = new HashSet<>();
        for (LinkedList<MapEntry<K, V>> bucket : buckets) {
            if (bucket == null) continue;
            for (MapEntry<K, V> pair : bucket) {
                set.add(pair);
            }
        }
        return set;
    }

    public V put(K key, V value) {
        V oldValue = null;
        int index = Math.abs(key.hashCode()) % SIZE;
        if (buckets[index] == null) {
            buckets[index] = new LinkedList<>();
        }
        MapEntry<K, V> pair = new MapEntry<>(key, value);
        LinkedList<MapEntry<K, V>> bucket = buckets[index];
        ListIterator<MapEntry<K, V>> iterator = bucket.listIterator();
        boolean hasfound = false;
        while (iterator.hasNext()) {
            MapEntry<K, V> iPair = iterator.next();
            if (iPair.getKey().equals(key)) {
                oldValue = iPair.getValue();
                hasfound = true;
                break;
            }
        }
        if (!hasfound) {
            buckets[index].add(new MapEntry<>(key, value));
        }
        return oldValue;

    }

    public V get(Object key) {
        int index = Math.abs(key.hashCode()) % 997;
        for (MapEntry<K, V> iPair : buckets[index]) {
            if (iPair.getKey() == key)
                return iPair.getValue();
        }
        return null;
    }

    public static void main(String[] args) {
        SimpleHashMap<String, String> map = new SimpleHashMap<>();
        map.put("key0", "value0");
        map.put("key1", "value1");
        System.out.println(map);
    }
}
```
```java
MapEntry.java
public class MapEntry<K, V> implements Map.Entry<K, V> {
    private K key;
    private V value;

    public MapEntry(K key, V value) {
        this.key = key;
        this.value = value;
    }

    @Override
    public K getKey() {
        return key;
    }

    @Override
    public V getValue() {
        return value;
    }

    @Override
    public V setValue(V v) {
        V oldValue = value;
        value = v;
        return oldValue;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof MapEntry)) return false;
        MapEntry me = (MapEntry) o;
        return (key == null ?
                me.getKey() == null : key.equals(me.getKey())) &&
                (value == null ?
                        me.getValue() == null : value.equals(me.getValue()));
    }

    @Override
    public int hashCode() {
        return (key == null ? 0 : key.hashCode()) ^ (value == null ? 0 : value.hashCode());
    }
}
```

