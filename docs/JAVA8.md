## stream

list<Class T> 转 Map<Object, List<Object>> 将集合中对象某一个字段抽为key，另一个字段的集合 为 value。

```java
Map<Object, List<Object>> map = list.stream().collect(Collectors.groupingBy(T::getField1, Collectors.mapping(T::getFeild2, Collectors.toList())));
```

list<Class T> 转 Map<Object, Class T> 将集合中对象某一个字段抽为 key， 对象为 value。

```
Mpa<Object, Class T> map = list.stream().collect(Collectors.toMap(key -> key.getField1(), value -> value));
```

参考链接:

[Java 8 简明教程](https://www.iteye.com/magazines/129-Java-8-Tutorial)

