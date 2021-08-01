# ArrayList Capacity

## java.util.ArrayList
```java
private static final int DEFAULT_CAPACITY = 10;

private int newCapacity(int minCapacity) {
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    ...
}
```

## Size & Capacity Test
> https://stackoverflow.com/questions/3564837/capacity-of-arraylist/3564855

```java
@Test
public void arrayTest() throws Exception {
    ArrayList<Integer> list = new ArrayList<>();
    for (int i = 0; i < 17; i++) {
        list.add(i);
        System.out.format("Size: %2d, Capacity: %2d%n",
                list.size(), getCapacity(list));
    }
}

private int getCapacity(ArrayList<?> l) throws Exception {
    Field dataField = ArrayList.class.getDeclaredField("elementData");
    dataField.setAccessible(true);
    return ((Object[]) dataField.get(l)).length;
}
```

### Output

```
Size:  1, Capacity: 10
Size:  2, Capacity: 10
Size:  3, Capacity: 10
Size:  4, Capacity: 10
Size:  5, Capacity: 10
Size:  6, Capacity: 10
Size:  7, Capacity: 10
Size:  8, Capacity: 10
Size:  9, Capacity: 10
Size: 10, Capacity: 10
Size: 11, Capacity: 15
Size: 12, Capacity: 15
Size: 13, Capacity: 15
Size: 14, Capacity: 15
Size: 15, Capacity: 15
Size: 16, Capacity: 22
Size: 17, Capacity: 22
```

> adoptopenjdk-8.jdk
