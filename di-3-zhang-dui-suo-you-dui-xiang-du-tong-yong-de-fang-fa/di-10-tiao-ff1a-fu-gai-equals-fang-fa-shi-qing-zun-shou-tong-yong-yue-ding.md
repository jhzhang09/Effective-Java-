### 第10条：覆盖equals方法时请遵守通用约定

Overriding the equals method seems simple, but there are many ways to get it wrong, and consequences can be dire. The easiest way to avoid problems is not to override the equals method, in which case each instance of the class is equal only to itself. This is the right thing to do if any of the following conditions apply:

* **Each instance of the class is inherently unique.** This is true for classes such as Thread that represent active entities rather than values. The equals implementation provided by Object has exactly the right behavior for these classes.
* **There is no need for the class to provide a “logical equality” test.** For example, java.util.regex.Pattern could have overridden equals to check whether two Pattern instances represented exactly the same regular expression, but the designers didn’t think that clients would need or want this functionality. Under these circumstances, the equals implementation inherited from Object is ideal.
* **A superclass has already overridden equals, and the superclass behavior is appropriate for this class.** For example, most Set implementations inherit their equals implementation from AbstractSet, List implementations from AbstractList, and Map implementations from AbstractMap.
* **The class is private or package-private, and you are certain that its equals method will never be invoked.** If you are extremely risk-averse, you can override the equals method to ensure that it isn’t invoked accidentally:

覆盖equals方法看起来好像挺简单的，但其实有许多覆盖方式会导致错误，并且会导致严重的后果。避免这些问题最简单的方式是不覆盖equals方法，在这种情况下，类的每个实例只跟它自己等价。如果满足以下任一条件，那么我们做的就是所期望的：

* **类的每个实例本质上都是独一无二的。**

```
@Override 
public boolean equals(Object o) {
    throw new AssertionError(); // Method is never called
}
```

So when is it appropriate to override equals? It is when a class has a

notion of logical equality that differs from mere object identity and

a superclass has not already overridden equals. This is generally the

case for value classes. A value class is simply a class that

represents a value, such as Integer or String. A programmer who

compares references to value objects using the equals method

expects to find out whether they are logically equivalent, not

whether they refer to the same object. Not only is overriding

the equals method necessary to satisfy programmer expectations, it

enables instances to serve as map keys or set elements with

predictable, desirable behavior.

One kind of value class that does not require the equals method to

be overridden is a class that uses instance control \(Item 1\) to

ensure that at most one object exists with each value. Enum types

\(Item 34\) fall into this category. For these classes, logical equality

is the same as object identity, so Object’s equals method functions

as a logical equals method.

When you override the equals method, you must adhere to its

general contract. Here is the contract, from the specification

for Object :

The equals method implements an equivalence relation. It has

these properties:

• Reflexive: For any non-null reference value x, x.equals\(x\) must

return true.

• Symmetric: For any non-null reference

values x and y, x.equals\(y\) must return true if and only

if y.equals\(x\) returns true.• Transitive: For any non-null reference values x, y, z,

if x.equals\(y\) returns true and y.equals\(z\) returns true,

then x.equals\(z\) must return true.

• Consistent: For any non-null reference values x and y, multiple

invocations of x.equals\(y\) must consistently return true or

consistently return false, provided no information used

in equals comparisons is modified.

• For any non-null reference value x, x.equals\(null\) must

return false.

Unless you are mathematically inclined, this might look a bit scary,

but do not ignore it! If you violate it, you may well find that your

program behaves erratically or crashes, and it can be very difficult

to pin down the source of the failure. To paraphrase John Donne,

no class is an island. Instances of one class are frequently passed to

another. Many classes, including all collections classes, depend on

the objects passed to them obeying the equals contract.

Now that you are aware of the dangers of violating

the equals contract, let’s go over the contract in detail. The good

news is that, appearances notwithstanding, it really isn’t very

complicated. Once you understand it, it’s not hard to adhere to it.

So what is an equivalence relation? Loosely speaking, it’s an

operator that partitions a set of elements into subsets whose

elements are deemed equal to one another. These subsets are

known as equivalence classes. For an equals method to be useful,

all of the elements in each equivalence class must be

interchangeable from the perspective of the user. Now let’s

examine the five requirements in turn:

Reflexivity—The first requirement says merely that an object

must be equal to itself. It’s hard to imagine violating this one

unintentionally. If you were to violate it and then add an instance

of your class to a collection, the contains method might well say

that the collection didn’t contain the instance that you just added.Symmetry—The second requirement says that any two objects

must agree on whether they are equal. Unlike the first requirement,

it’s not hard to imagine violating this one unintentionally. For

example, consider the following class, which implements a

case-insensitive string. The case of the string is preserved

by toString but ignored in equals comparisons:

```
// Broken - violates symmetry!
public final class CaseInsensitiveString {
private final String s;
public CaseInsensitiveString(String s) {
this.s = Objects.requireNonNull(s);
} /
/ Broken - violates symmetry!
@Override public boolean equals(Object o) {
if (o instanceof CaseInsensitiveString)
return s.equalsIgnoreCase(
((CaseInsensitiveString) o).s);
if (o instanceof String) // One-way interoperability!
return s.equalsIgnoreCase((String) o);
return false;
} ..
. // Remainder omitted
}
```

The well-intentioned equals method in this class naively attempts

to interoperate with ordinary strings. Let’s suppose that we have

one case-insensitive string and one ordinary one:

```
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```

As expected, cis.equals\(s\) returns true. The problem is that while

the equals method in CaseInsensitiveString knows about ordinary

strings, the equals method in String is oblivious to case-insensitive

strings. Therefore, s.equals\(cis\) returns false, a clear violation of

symmetry. Suppose you put a case-insensitive string into a

collection:

```
List<CaseInsensitiveString> list = new ArrayList<>();
list.add(cis);
```

What does list.contains\(s\) return at this point? Who knows? In the

current OpenJDK implementation, it happens to return false, but

that’s just an implementation artifact. In another implementation,

it could just as easily return true or throw a runtime

exception. Once you’ve violated the equals contract, you

simply don’t know how other objects will behave when

confronted with your object.

To eliminate the problem, merely remove the ill-conceived attempt

to interoperate with String from the equals method. Once you do

this, you can refactor the method into a single return statement:

```
@Override public boolean equals(Object o) {
return o instanceof CaseInsensitiveString &&
((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

Transitivity—The third requirement of the equals contract says

that if one object is equal to a second and the second object is equal

to a third, then the first object must be equal to the third. Again,it’s not hard to imagine violating this requirement unintentionally.

Consider the case of a subclass that adds a new value

component to its superclass. In other words, the subclass adds a

piece of information that affects equals comparisons. Let’s start

with a simple immutable two-dimensional integer point class:

```
public class Point {
private final int x;
private final int y;
public Point(int x, int y) {
this.x = x;
this.y = y;
} @
Override public boolean equals(Object o) {
if (!(o instanceof Point))
return false;
Point p = (Point)o;
return p.x == x && p.y == y;
} ..
. // Remainder omitted
}
```

Suppose you want to extend this class, adding the notion of color to

a point:

```
public class ColorPoint extends Point {
private final Color color;public ColorPoint(int x, int y, Color color) {
super(x, y);
this.color = color;
} ..
. // Remainder omitted
}
```

How should the equals method look? If you leave it out entirely, the

implementation is inherited from Point and color information is

ignored in equals comparisons. While this does not violate

the equals contract, it is clearly unacceptable. Suppose you write

an equals method that returns trueonly if its argument is another

color point with the same position and color:

```
// Broken - violates symmetry!
@Override public boolean equals(Object o) {
if (!(o instanceof ColorPoint))
return false;
return super.equals(o) && ((ColorPoint) o).color ==
color;
}
```

The problem with this method is that you might get different

results when comparing a point to a color point and vice versa. The

former comparison ignores color, while the latter comparison

always returns false because the type of the argument is incorrect.

To make this concrete, let’s create one point and one color point:

```
Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);
```

Then p.equals\(cp\) returns true, while cp.equals\(p\) returns false. You

might try to fix the problem by having ColorPoint.equals ignore

color when doing “mixed comparisons”:

```
// Broken - violates transitivity!
@Override public boolean equals(Object o) {
if (!(o instanceof Point))
return false;
// If o is a normal Point, do a color-blind comparison
if (!(o instanceof ColorPoint))
return o.equals(this);
// o is a ColorPoint; do a full comparison
return super.equals(o) && ((ColorPoint) o).color == color;
}
```

This approach does provide symmetry, but at the expense of

transitivity:

```
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```

Now p1.equals\(p2\) and p2.equals\(p3\) return true,

while p1.equals\(p3\) returns false, a clear violation of transitivity.

The first two comparisons are “color-blind,” while the third takes

color into account.

Also, this approach can cause infinite recursion: Suppose there are

two subclasses of Point, say ColorPoint and SmellPoint, each with thissort of equals method. Then a call

to myColorPoint.equals\(mySmellPoint\) will throw a StackOverflowError.

So what’s the solution? It turns out that this is a fundamental

problem of equivalence relations in object-oriented

languages. There is no way to extend an instantiable class

and add a value component while preserving

the equals contract, unless you’re willing to forgo the benefits of

object-oriented abstraction.

You may hear it said that you can extend an instantiable class and

add a value component while preserving the equals contract by

using a getClass test in place of the instanceof test in

the equals method:

```
// Broken - violates Liskov substitution principle (page 43)
@Override public boolean equals(Object o) {
if (o == null || o.getClass() != getClass())
return false;
Point p = (Point) o;
return p.x == x && p.y == y;
}
```

This has the effect of equating objects only if they have the same

implementation class. This may not seem so bad, but the

consequences are unacceptable: An instance of a subclass

of Point is still a Point, and it still needs to function as one, but it

fails to do so if you take this approach! Let’s suppose we want to

write a method to tell whether a point is on the unit circle. Here is

one way we could do it:

```
// Initialize unitCircle to contain all Points on the unit
circleprivate static final Set<Point> unitCircle = Set.of(
new Point( 1, 0), new Point( 0, 1),
new Point(-1, 0), new Point( 0, -1));
public static boolean onUnitCircle(Point p) {
return unitCircle.contains(p);
}
```

While this may not be the fastest way to implement the

functionality, it works fine. Suppose you extend Point in some

trivial way that doesn’t add a value component, say, by having its

constructor keep track of how many instances have been created:

```
public class CounterPoint extends Point {
private static final AtomicInteger counter =
new AtomicInteger();
public CounterPoint(int x, int y) {
super(x, y);
counter.incrementAndGet();
} p
ublic static int numberCreated() { return counter.get(); }
}
```


