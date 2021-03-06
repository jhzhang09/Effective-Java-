### 第18条：组合优先于继承

Inheritance is a powerful way to achieve code reuse, but it is not always the best tool for the job. Used inappropriately, it leads to fragile software. It is safe to use inheritance within a package, where the subclass and the superclass implementations are under the control of the same programmers. It is also safe to use inheritance when extending classes specifically designed and documented for extension \(Item 19\). Inheriting from ordinary concrete classes across package boundaries, however, is dangerous. As a reminder, this book uses the word “inheritance” to mean implementation inheritance \(when one class extends another\). The problems discussed in this item do not apply to interface inheritance\(when a class implements an interface or when one interface extends another\).

继承（inheritance）是代码复用的一种有效途径，但不是最佳方式。使用不当的话，容易使做出来的软件变得脆弱。在同一个包里使用继承是安全的，这样的话子类实现和父类实现都在同个程序员的控制之下。除此之外，若类是专门设计来被继承而且具有良好文档话，则采用继承来进行扩展也是安全的。但是，如果只是跨包继承一个普通具体的类，则会有危险。说明一下，本书采用“继承”来指实现继承（当一个类扩展了另一个类）。本条目讨论的问题不适用于接口继承（当一个类实现了一个接口或者一个接口扩展了另一个接口）。

**Unlike method invocation, inheritance violates encapsulation** \[Snyder86\]. In other words, a subclass depends on the implementation details of its superclass for its proper function. The superclass’s implementation may change from release to release, and if it does, the subclass may break, even though its code has not been touched. As a consequence, a subclass must evolve in tandem with its superclass, unless the superclass’s authors have designed and documented it specifically for the purpose of being extended.

**与方法调用不同的是，继承违反了封装原则**\[Snyder86\]。换句话说，一个子类依赖于它的父类的实现细节来实现它本身的功能。但随着版本的更新，父类的实现会产生变化，一旦这种情况发生了，子类将会被破坏，即便子类的代码没有变更过。这样的后果是，子类必须跟着父类一起演化，除非编写父类的作者本来就是为了专门将其设计成被继承，同时还准备了良好的文档。

To make this concrete, let’s suppose we have a program that uses a HashSet. To tune the performance of our program, we need to query the HashSet as to how many elements have been added since it was created \(not to be confused with its current size, which goes down when an element is removed\). To provide this functionality, we write a HashSet variant that keeps count of the number of attempted element insertions and exports an accessor for this count. The HashSet class contains two methods capable of adding elements, add and addAll, so we override both of these methods:

为了更具体地说明这一点，我们假设有一个程序，这个程序用了HashSet。为了调优程序的性能，我们需要查一下这个HashSet自从被创建以来添加了多少个元素进去（不要跟当前的大小混淆起来，元素的数目会随着删除而减少）。为了实现这个功能，我们写了一个HashSet变量，它记录着尝试插入的元素的数目，同时为这个数目提供一个访问方法。HashSet类包含了两个用来添加元素的方法，add和addAll，所以我们覆盖这两个方法：

```
// Broken - Inappropriate use of inheritance!
public class InstrumentedHashSet<E> extends HashSet<E> {
    // The number of attempted element insertions
    private int addCount = 0;
    public InstrumentedHashSet() {
    } 
    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    } 
    @Override 
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    } 
    @Override 
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    } 
    public int getAddCount() {
        return addCount;
    }
}
```

This class looks reasonable, but it doesn’t work. Suppose we create an instance and add three elements using the addAll method. Incidentally, note that we create a list using the static factory method List.of, which was added in Java 9; if you’re using an earlier release, use Arrays.asList instead:

上述这个类看起来是合理的，但它无法工作。假设我们创建了一个实例并通过addAll方法添加了三个元素进去。顺便说一下，注意到我们创建列表时，使用了Java 9就开始添加进来的静态工厂方法List.of。如果你使用了更早的JDK版本，请用Arrays.asList来替换：

```
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("Snap", "Crackle", "Pop"));
```

We would expect the getAddCount method to return three at this point, but it returns six. What went wrong? Internally, HashSet’s addAll method is implemented on top of its add method, although HashSet, quite reasonably, does not document this implementation detail. The addAll method in InstrumentedHashSet added three to addCount and then invoked HashSet’s addAll implementation using super.addAll. This in turn invoked the add method, as overridden in InstrumentedHashSet, once for each element. Each of these three invocations added one more to addCount, for a total increase of six: each element added with the addAll method is double-counted.

此时，我们期待着getAddCount方法会返回3，但实际上它返回了6。哪里出错了？在HashSet内部，addAll方法是基于它的add方法来实现的，虽然HashSet并没有在文档里说明这个实现细节，但这是合理的。InstrumentedHashSet的addAll方法对addCount加3，然后再通过super.addAll语句调用HashSet的addAll方法。但调用HashSet的addAll方法时又去调用在InstrumentedHashSet类中重写的add方法，每个元素调用一次。这三次调用每次都将addCount加1，最终addCount总共加了6：add方法加了3次1，addAll方法加了3，也就是每个元素都算了两次。

We could “fix” the subclass by eliminating its override of the addAll method. While the resulting class would work, it would depend for its proper function on the fact that HashSet’s addAll method is implemented on top of its add method. This “self-use” is an implementation detail, not guaranteed to hold in all implementations of the Java platform and subject to change from release to release. Therefore, the resulting InstrumentedHashSet class would be fragile.

我们只要去掉覆盖的addAll方法，就可以“修复”这个问题。虽然这样做之后类可以工作了，但它的正常运作却依赖于这么一个事实：HashSet的addAll方法是基于它的add方法的。这种“自用性（self-use）”是一种实现细节，并不保证在Java平台的所有实现中都是这样，而且受制于版本之间的不同。因此，这样编写出来的InstrumentedHashSet类将是脆弱的。

It would be slightly better to override the addAll method to iterate over the specified collection, calling the add method once for each element. This would guarantee the correct result whether or not HashSet’s addAll method were implemented atop its add method because HashSet’s addAll implementation would no longer be invoked. This technique, however, does not solve all our problems. It amounts to reimplementing superclass methods that may or may not result in self-use, which is difficult, time-consuming, error-prone, and may reduce performance. Additionally, it isn’t always possible because some methods cannot be implemented without access to private fields inaccessible to the subclass.

稍微好点的做法是，覆盖addAll方法，在方法内部遍历指定的集合，对每个元素都调用add方法。这样做可以保证，无论HashSet的addAll方法的实现是否依赖于add方法，都会得到正确的结果，因为HashSet的addAll实现不会被调用。然而，这种技术并不能解决我们所有的问题。这么做相当于重新实现一遍父类的方法，这些父类方法还不知是不是自用（self-use）的，而且。重新实现一遍不仅有难度，耗时，容易出错，而且还有可能还会降低性能。不仅如此，有时可能还无法这么做，因为有些方法不能在子类里访问某些父类的私有属性，从而导致其不能被实现。

A related cause of fragility in subclasses is that their superclass can acquire new methods in subsequent releases. Suppose a program depends for its security on the fact that all elements inserted into some collection satisfy some predicate. This can be guaranteed by subclassing the collection and overriding each method capable of adding an element to ensure that the predicate is satisfied before adding the element. This works fine until a new method capable of inserting an element is added to the superclass in a subsequent release. Once this happens, it becomes possible to add an “illegal” element merely by invoking the new method, which is not overridden in the subclass. This is not a purely theoretical problem. Several security holes of this nature had to be fixed when Hashtable and Vector were retrofitted to participate in the Collections Framework.

导致子类脆弱的一个相关原因是它们的父类在后续的版本中添加新的方法。假设一个程序的安全性依赖于这么一个事实：所有插入集合的元素都要满足一些先决条件。下面这种做法可以确保这一点：对集合子类化，同时覆盖每一个可以添加元素的方法来保证该元素被添加之前是满足指定先决条件的。这么做没问题，但是当父类在接下来的版本中添加了一个新的能插入元素的方法时，问题就来了。一旦这种情况发生了，将有可能仅是调用这个未被覆盖的新方法就把一个“不合法”的元素添加进去了。这不只是个单纯的理论问题。在Hashtable和Vector在加入集合框架（Collections Framework）时，就修复了这类性质的安全漏洞。

Both of these problems stem from overriding methods. You might think that it is safe to extend a class if you merely add new methods and refrain from overriding existing methods. While this sort of extension is much safer, it is not without risk. If the superclass acquires a new method in a subsequent release and you have the bad luck to have given the subclass a method with the same signature and a different return type, your subclass will no longer compile \[JLS, 8.4.8.3\]. If you’ve given the subclass a method with the same signature and return type as the new superclass method, then you’re now overriding it, so you’re subject to the problems described earlier. Furthermore, it is doubtful that your method will fulfill the contract of the new superclass method, because that contract had not yet been written when you wrote the subclass method.

上述这两个问题（子类实现依赖于父类实现；父类添加方法了还有可能出现漏洞）都来源于覆盖方法。你可能会觉得，如果仅仅是添加新方法同时避免覆盖现有方法将会是安全的。虽然这种扩展会安全很多，但它也并不是没有风险的。如果父类在将来的版本中要添加一个新的方法，而你又恰好在子类里写了一个相同签名但返回类型不同的方法，那么你的子类将编译不通过\[JLS, 8.4.8.3\]。如果你写了一个签名和返回类型都与父类的新方法一样的方法，那么你就是在覆盖这个新方法，这样你就会遇到前面描述到的问题。此外，你的方法是否满足了新的父类方法的约定这一点也是值得怀疑的，因为你在写子类方法的时候父类新方法的约定还没确定。

Luckily, there is a way to avoid all of the problems described above. Instead of extending an existing class, give your new class a private field that references an instance of the existing class. This design is called composition because the existing class becomes a component of the new one. Each instance method in the new class invokes the corresponding method on the contained instance of the existing class and returns the results. This is known as forwarding, and the methods in the new class are known as forwarding methods. The resulting class will be rock solid, with no dependencies on the implementation details of the existing class. Even adding new methods to the existing class will have no impact on the new class. To make this concrete, here’s a replacement for InstrumentedHashSet that uses the composition-and-forwarding approach. Note that the implementation is broken into two pieces, the class itself and a reusable forwarding class, which contains all of the forwarding methods and nothing else:

幸运的是，有一个办法可以避免所有上面说到的问题。除了扩展现有类，你还可以让你的新类包含一个私有域，这个域指向现有类的一个实例。这种设计被称为组合，因为现有类成为了新类组件。新类的每个实例方法调用现有类实例的对应方法然后返回值。这种做法被称为转发，新类里的方法也被称为转发方法。这样编写出来的类将会很坚固，因为它不依赖域现有类的实现细节。即使往现有类里添加新方法也不影响新类。为了更具体地说明，下面采用“组合与转发”的方式写一个InstrumentedHashSet的替代实现。注意，新的实现方式分为两部分，类本身和一个可复用的转发类。转发类仅包含所有的转发方法：

```
// Wrapper class - uses composition in place of inheritance
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;
    public InstrumentedSet(Set<E> s) {
        super(s);
    }
    @Override 
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    } 
    @Override 
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    } 
    public int getAddCount() {
        return addCount;
    }
} 
// Reusable forwarding class
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { 
        this.s = s; 
    }
    public void clear() { 
        s.clear(); 
    }
    public boolean contains(Object o) { 
        return s.contains(o); 
    }
    public boolean isEmpty() { 
        return s.isEmpty(); 
    }
    public int size() { 
        return s.size(); 
    }
    public Iterator<E> iterator() { 
        return s.iterator(); 
    }
    public boolean add(E e) { 
        return s.add(e); 
    }
    public boolean remove(Object o) { 
        return s.remove(o); 
    }
    public boolean containsAll(Collection<?> c) { 
        return s.containsAll(c); 
    }
    public boolean addAll(Collection<? extends E> c) { 
        return s.addAll(c); 
    }
    public boolean removeAll(Collection<?> c) { 
        return s.removeAll(c); 
    }
    public boolean retainAll(Collection<?> c) { 
        return s.retainAll(c); 
    }
    public Object[] toArray() { 
        return s.toArray(); 
    }
    public <T> T[] toArray(T[] a) { 
        return s.toArray(a); 
    }
    @Override 
    public boolean equals(Object o) { 
        return s.equals(o); 
    }
    @Override 
    public int hashCode() { 
        return s.hashCode(); 
    }
    @Override 
    public String toString() { 
        return s.toString(); 
    }
}
```

The design of the InstrumentedSet class is enabled by the existence of the Set interface, which captures the functionality of the HashSet class. Besides being robust, this design is extremely flexible. The InstrumentedSet class implements the Set interface and has a single constructor whose argument is also of type Set. In essence, the class transforms one Set into another, adding the instrumentation functionality. Unlike the inheritance-based approach, which works only for a single concrete class and requires a separate constructor for each supported constructor in the superclass, the wrapper class can be used to instrument any Set implementation and will work in conjunction with any preexisting constructor:

InstrumentedSet类的设计依赖于现有的Set接口，Set接口里包含着HashSet类的方法。这么设计不但健壮，而且很灵活。InstrumentedSet类实现了Set接口，同时只有一个构造器，这个构造器要求传入一个Set类型的参数。本质上，这个类其实是将一个Set转换成另一个，并添加一些自己的功能。前面讨论到的基于继承的方式只能为一个具体的类工作，而且必须分别为父类里的每个构造器提供一个相应的构造器，而上述例子中的包装类能用于任意类型的Set实现，同时能与任意现存的构造器共存。

```
Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

The InstrumentedSet class can even be used to temporarily instrument a set instance that has already been used without instrumentation:

InstrumentedSet类甚至可以临时用于原本就没有计数特性的Set实例：

```
static void walk(Set<Dog> dogs) {
    InstrumentedSet<Dog> iDogs = new InstrumentedSet<>(dogs);
    ... // Within this method use iDogs instead of dogs
}
```

The InstrumentedSet class is known as a wrapper class because each InstrumentedSet instance contains \(“wraps”\) another Set instance. This is also known as the Decorator pattern \[Gamma95\] because the InstrumentedSet class “decorates” a set by adding instrumentation. Sometimes the combination of composition and forwarding is loosely referred to as delegation. Technically it’s not delegation unless the wrapper object passes itself to the wrapped object \[Lieberman86; Gamma95\].

InstrumentedSet类之所以被称为包装类，是因为每个InstrumentedSet实例都包装了另一个Set实例。这种模式也被称为装饰者模式\[Gamma95\]因为InstrumentedSet类通过添加计数功能“装饰”了一个Set实例。有时组合和转发放在一起也会被简单地称为代理。从技术角度来说，这并不是代理，除非包装者对象（wrapper class）将其自己传给被包装对象\[Lieberman86; Gamma95\]。

The disadvantages of wrapper classes are few. One caveat is that wrapper classes are not suited for use in callback frameworks, where in objects pass self-references to other objects for subsequent invocations \(“callbacks”\). Because a wrapped object doesn’t know of its wrapper, it passes a reference to itself \(this\) and callbacks elude the wrapper. This is known as the SELF problem \[Lieberman86\]. Some people worry about the performance impact of forwarding method invocations or the memory footprint impact of wrapper objects. Neither turn out to have much impact in practice. It’s tedious to write forwarding methods, but you have to write the reusable forwarding class for each interface only once, and forwarding classes may be provided for you. For example, Guava provides forwarding classes for all of the collection interfaces \[Guava\].

包装者对象几乎没有缺点。要提的一个警告是，包装者对象不适用于回调框架，因为在回调框架里，对象需要将自身引用传给别的对象，以便别的对象在后续进行调用（“回调”）。因为被包装对象并不知道它的包装者，它将一个引用传给它自己（this）同时回调也避开了包装者。这被称为SELF问题\[Lieberman86\]。一些人可能会担心转发方法调用的性能影响或者包装者对象（wrapper object）的内存占用。在实际应用中，这两个问题都不会有太大影响。虽然转发方法写起来会有点乏味，但你只需为每个接口写一次可复用的转发类。而且，有时转发类可能已经提供给你了，例如，Guava就为所有的集合接口提供了转发类\[Guava\]。

Inheritance is appropriate only in circumstances where the subclass really is a subtype of the superclass. In other words, a class B should extend a class A only if an “is-a” relationship exists between the two classes. If you are tempted to have a class B extend a class A, ask yourself the question: Is every B really an A? If you cannot truthfully answer yes to this question, B should not extend A. If the answer is no, it is often the case that B should contain a private instance of A and expose a different API: A is not an essential part of B, merely a detail of its implementation. There are a number of obvious violations of this principle in the Java platform libraries. For example, a stack is not a vector, so Stack should not extend Vector. Similarly, a property list is not a hash table, so Properties should not extend Hashtable. In both cases, composition would have been preferable.

继承只适用于一个类的类型的确是某个父类的子类型的情况。换句话说，只有当类B和类A是“is-a”的关系时，类B才应该扩展类A。如果你打算让类B扩展类A，可以自问一下这个问题：是否每个B都确实是一个A？如果你对这个问题无法肯定地回答yes，那么B就不应该扩展A。如果答案是no，那么通常是B应该包含一个私有的A的实例，同时暴露一个不同的API：A并不是B的一个重要组成部分，而仅仅是B的实现的一个细节。在Java类库里有好几个地方明显违反了这个原则。例如，一个stack并不是vector，所以Stack不应该继承自Vector。类似地，一个属性列表（property list）并不是一个哈希表，所以Properties不应该继承自Hashtable。在这两种场景中，使用组合会更恰当点。

If you use inheritance where composition is appropriate, you needlessly expose implementation details. The resulting API ties you to the original implementation, forever limiting the performance of your class. More seriously, by exposing the internals you let clients access them directly. At the very least, it can lead to confusing semantics. For example, if p refers to a Properties instance, then p.getProperty\(key\) may yield different results from p.get\(key\): the former method takes defaults into account, while the latter method, which is inherited from Hashtable, does not. Most seriously, the client may be able to corrupt invariants of the subclass by modifying the superclass directly. In the case of Properties, the designers intended that only strings be allowed as keys and values, but direct access to the underlying Hashtable allows this invariant to be violated. Once violated, it is no longer possible to use other parts of the Properties API \(load and store\). By the time this problem was discovered, it was too late to correct it because clients depended on the use of non-string keys and values.

如果你在使用组合更恰当的地方使用了继承，那么你会不必要地去暴露实现细节。这样得到的API会将你限制于原始的实现，并永远限制了你的类的性能。更严重的是，客户端可以通过直接修改父类来破坏子类的约束条件。以Properties为例，设计者意图只有字符串可以被允许作为键和值，但直接访问底层的Hashtable则可能会违反这个意图假设。一旦返回了，Properties的其它API就无法再使用（load方法和store方法）。在发现这个问题时并想要去修复的时候已经太晚了，因为很多客户端已经依赖于非字符串的键和值的使用。

There is one last set of questions you should ask yourself before deciding to use inheritance in place of composition. Does the class that you contemplate extending have any flaws in its API? If so, are you comfortable propagating those flaws into your class’s API? Inheritance propagates any flaws in the superclass’s API, while composition lets you design a new API that hides these flaws.

在决定使用继承而不是组合之前，你应该先问问自己一些问题。你准备继承的那个类在API上是否有一些缺陷？如果有，你是否愿意将那些缺陷传播到你的类的API里？继承会传播父类API里的缺陷，而组合却能让你设计一个能隐藏这些缺陷的新的API。

To summarize, inheritance is powerful, but it is problematic because it violates encapsulation. It is appropriate only when a genuine subtype relationship exists between the subclass and the superclass. Even then, inheritance may lead to fragility if the subclass is in a different package from the superclass and the superclass is not designed for inheritance. To avoid this fragility, use composition and forwarding instead of inheritance, especially if an appropriate interface to implement a wrapper class exists. Not only are wrapper classes more robust than subclasses, they are also more powerful.

总的来说，继承虽然强大，但它存在一些问题，因为它违反了封装。它只有在子类和父类之间存在真正的子类型关系的时候才使用。即便这样，如果子类在一个与父类不同的包中而且父类本来就不是设计来被继承的，那么继承将会导致子类的脆弱性。为了避免这种脆弱性，我们应该使用组合与转发，而不是继承，尤其是存在一个适当的接口来实现一个现存的包装者类。包装者类不仅比子类更健壮，而且更强大。

