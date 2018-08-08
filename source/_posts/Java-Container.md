---
title: Java-Container
date: 2018/7/30
categories: Java
---
### Basic Concept
Container is for storing objects. Container has two types.
 - *Collection: An independent sequence.Such as List,Set,Queue.*
 - *Map: A paired object of key and value.* 
### Difference between List and Set and Map
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
### List
 - ArrayList: Good at querying data randomly. Poor at  insert or delete elements.
 - LinkedList : Store elements by added sequence.Good at inserting or deleting elements frequently.

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
### Set
 - HashSet: HashSet is the fastest way in getting elements.
 - TreeSet: TreeSet stores elements in ascending. 
 - LinkedHashSet : LinkedHashSet store elements by added sequence.
 - SortedSet: An interface implemented by TreeSet.

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
### Map
 - HashMap: HashMap is the fastest way in finding elements.
 - TreeMap:TreeMap stored element in ascending.
 - LinkerHashMap: LinkedHashMap store elements by added sequence.



### Something about Array.asList
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
