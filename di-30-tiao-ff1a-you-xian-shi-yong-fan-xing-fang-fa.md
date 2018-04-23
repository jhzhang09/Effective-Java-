### 第30条：优先使用泛型方法

Just as classes can be generic, so can methods. Static utility methods that operate on parameterized types are usually generic. All of the “algorithm” methods in Collections \(such as binarySearch and sort\) are generic.

就像类可以是泛型的那样，方法也是可以的。作用与参数化类型的静态工具方法通常都是泛型的。Collections类的所有“算法”方法（如binarySearch方法和sort方法）都是泛型的。

Writing generic methods is similar to writing generic types. Consider this deficient method, which returns the union of two sets:

编写泛型方法类似于编写泛型类型。考虑下面这个有缺陷的方法，它返回了两个集合的并集：

```
// Uses raw types - unacceptable! (Item 26)
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

This method compiles but with two warnings:

这个方法可以编译通过，但会出现两条警告：

```
Union.java:5: warning: [unchecked] unchecked call to
HashSet(Collection<? extends E>) as a member of raw type
HashSet
Set result = new HashSet(s1);
^
Union.java:6: warning: [unchecked] unchecked call to
addAll(Collection<? extends E>) as a member of raw type Set
result.addAll(s2);
^
```

To fix these warnings and make the method typesafe, modify its declaration to declare a type parameter representing the element type for the three sets \(the two arguments and the return value\) and use this type parameter throughout the method.** The type parameter list, which declares the type parameters, goes between a method’s modifiers and its return type.** In this example, the type parameter list is &lt;E&gt;, and the return type is Set&lt;E&gt;. The naming conventions for type parameters are the same for generic methods and generic types \(Items 29, 68\):

为了修复这些警告并使得这个方法类型安全，我们可以修改方法声明，即声明一个类型参数来表示三个集合（两个参数和返回值）的元素类型，同时在整个方法体内都使用这个类型参数。**声明了类型参数的类型参数列表在方法修饰符和它的返回类型之间。**在这个例子中，类型参数列表是&lt;E&gt;，返回类型是Set&lt;E&gt;。类型参数的命名规则跟泛型方法和泛型类型一致（条目29，68）：

```
// Generic method
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

At least for simple generic methods, that’s all there is to it. This

method compiles without generating any warnings and providestype safety as well as ease of use. Here’s a simple program to

exercise the method. This program contains no casts and compiles

without errors or warnings:
