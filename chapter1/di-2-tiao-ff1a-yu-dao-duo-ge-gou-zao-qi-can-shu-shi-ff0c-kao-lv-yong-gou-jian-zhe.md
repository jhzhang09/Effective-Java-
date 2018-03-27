### 第2条：遇到多个构造器参数时，考虑用构建者

Static factories and constructors share a limitation: they do not scale well to large numbers of optional parameters. Consider the case of a class representing the Nutrition Facts label that appears on packaged foods. These labels have a few required fields—serving size, servings per container, and calories per serving—and more than twenty optional fields—total fat, saturated fat, trans fat, cholesterol, sodium, and so on. Most products have nonzero values for only a few of these optional fields.

What sort of constructors or static factories should you write for such a class? Traditionally, programmers have used the telescoping constructor pattern, in which you provide a constructor with only the required parameters, another with a single optional parameter, a third with two optional parameters, and so on, culminating in a constructor with all the optional parameters. Here’s how it looks in practice. For brevity’s sake, only four optional fields are shown:

静态工厂和构造器都有个共同的不足的地方：它们都不能很好地扩展到大量的可选参数。考虑这么一种情况，用一个类来表示包装食品外面的营养成分标签。这些标签里有几个属性是必须有的：每份的含量，每罐的含量以及每份的卡路里，同时还有20个可选的属性：总脂肪含量、饱和脂肪含量、胆固醇含量、钠含量等等。大多数只在某几个可选属性里会有非零值。

对于这样的类，我们应该如何编写构造器或者静态工厂？一般情况下，程序员会习惯于用可伸缩构造器（telescoping constructor pattern）模式。在这种模式下，程序员会先提供一个只有必要参数的构造器，然后在这个构造器的基础上，提供一个还要需要有一个可选参数的构造器，接着提供一个需要有两个可选参数的构造器，以此类推，终于在最后一个构造器的参数列表里，不仅包含了那几个必要的参数，还包含了所有的可选参数。下面有个例子。为了简单起见，它只显示4个可选属性：

```
// Telescoping constructor pattern - does not scale well!
public class NutritionFacts {
    private final int servingSize; // (mL) required 
    private final int servings;    // (per container) required
    private final int calories;    // (per serving) optional    
    private final int fat;         // (g/serving) optional
    private final int sodium;      // (mg/serving) optional
    private final int carbohydrate; // (g/serving) optional
    public NutritionFacts(int servingSize, int servings) { 
        this(servingSize, servings, 0);
    }
    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0); 
    }
    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0); 
    }
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0); 
    }
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, 
        int carbohydrate) {
        this.servingSize = servingSize; this.servings = servings;
        this.calories = calories
        this.fat = fat
        this.sodium = sodium
        this.carbohydrate = carbohydrate;
    } 
}
```

When you want to create an instance, you use the constructor with the shortest parameter list containing all the parameters you want to set:

当你想创建一个实例时，就用那个包含了所有你想设置的参数而且是参数列表最短的那个构造器：

```
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

Typically this constructor invocation will require many parameters that you don’t want to set, but you’re forced to pass a value for them anyway. In this case, we passed a value of 0 for fat. With “only” six parameters this may not seem so bad, but it quickly gets out of hand as the number of parameters increases.

这个构造器的调用通常需要很多你原本不想设置的参数，但你还是不得不给这些参数传一个值进去。在上述的例子中，我们为参数fat传了0值进去。若仅仅是这6个参数，情况还好点，但随着参数的增多，很快就开始失控。

In short, **the telescoping constructor pattern works, but it is hard to write client code when there are many parameters, and harder still to read it.** The reader is left wondering what all those values mean and must carefully count parameters to find out. Long sequences of identically typed parameters can cause subtle bugs. If the client accidentally reverses two such parameters, the compiler won’t complain, but the program will misbehave at runtime \(Item 51\).

简而言之，**可伸缩构造器是可行，只是当有很多参数时，会让客户端代码很难编写，而且代码也很难阅读。**读者若想知道传入的那些值代表什么，就必须得仔细地数着这些参数来一探究竟。一长串相同类型的参数还会导致一些难以察觉的错误。假如客户端不小心将两个相同类型的参数调换来位置，编译器不会报错，但程序在运行时就不会按照我们所想的去做了（条目51）。

A second alternative when you’re faced with many optional parameters in a constructor is the JavaBeans pattern, in which you call a parameterless constructor to create the object and then call setter methods to set each required parameter and each optional parameter of interest:

对于这种一个构造器里有很多可选参数的情况，另一种可选的方案就是采用JavaBeans模式。若采用这种模式，则先调用无参构造器来创建一个对象，然后分别调用不同的setter方法来设置必要参数和可选参数：

```
// JavaBeans Pattern - allows inconsistency, mandates mutability
public class NutritionFacts {
    // Parameters initialized to default values (if any)
    private int servingSize = -1; // Required; no default value 
    private int servings = -1; // Required; no default value
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;
    public NutritionFacts() { }
    // Setters
    public void setServingSize(int val) { 
        servingSize = val; 
    } 
    public void setServings(int val) { 
        servings = val; 
    }
    public void setCalories(int val) {
        calories = val;
    }
    public void setFat(int val) {
        fat = val;
    }
    public void setSodium(int val) {
        sodium = val;
    }
    public void setCarbohydrate(int val) { 
        carbohydrate = val; 
    }
}
```

This pattern has none of the disadvantages of the telescoping constructor pattern. It is easy, if a bit wordy, to create instances, and easy to read the resulting code:

这种模式没有任一可伸缩构造器模式的缺点。说得明白一点，Java Beans创建实例简单，而且代码易于阅读：

```
NutritionFacts cocaCola = new NutritionFacts(); 
cocaCola.setServingSize(240); 
cocaCola.setServings(8); 
cocaCola.setCalories(100); 
cocaCola.setSodium(35); 
cocaCola.setCarbohydrate(27);
```

Unfortunately, the JavaBeans pattern has serious disadvantages of its own. Because construction is split across multiple calls, **a JavaBean may be in an inconsistent state partway through its construction.**The class does not have the option of enforcing consistency merely by checking the validity of the constructor parameters. Attempting to use an object when it’s in an inconsistent state may cause failures that are far removed from the code containing the bug and hence difficult to debug. A related disadvantage is that **the JavaBeans pattern precludes the possibility of making a class immutable**\(Item 17\) and requires added effort on the part of the programmer to ensure thread safety.

不幸的是，JavaBeans模式也有一些严重的缺陷。由于构造过程被分到了多个调用中，**一个JavaBean在其构造过程中可能处于不一致的状态。**类无法仅仅通过检查构造器参数的有效性来保证一致性。试图使用一个处于不一致状态的对象将会导致失败，而且这种失败远不像那些包含bug的代码，因此它调试起来非常困难。与此相关的另一个缺点是，JavaBeans模式阻止了把类做成不可变的可能性，这需要程序员付出额外的努力来确保线程安全。

It is possible to reduce these disadvantages by manually “freezing” the object when its construction is complete and not allowing it to be used until frozen, but this variant is unwieldy and rarely used in practice. Moreover, it can cause errors at runtime because the compiler cannot ensure that the programmer calls the freeze method on an object before using it.

当然，为了弥补这些不足，我们可以在对象初始化完成的时候手工将它冻结，然后在冻结之前都不允许它被使用，但这种方式很不灵活，而且在实践中也很少用这种方式。不仅如此，这种做法也容易引起运行时错误，因为编译器无法确保程序员在用这个对象之前调用它的冻结方法。

Luckily, there is a third alternative that combines the safety of the telescoping constructor pattern with the readability of the JavaBeans pattern. It is a form of the \_Builder \_pattern \[Gamma95\].

好在还有第三种方案，而且这种方案结合了可伸缩构造器模式的安全性和JavaBeans模式的可阅读性。它就是_Builder_模式。

Instead of making the desired object directly, the client calls a constructor \(or static factory\) with all of the required parameters and gets a builder object. Then the client calls setter-like methods on the builder object to set each optional parameter of interest. Finally, the client calls a parameterless build method to generate the object, which is typically immutable. The builder is typically a static member class \(Item 24\) of the class it builds. Here’s how it looks in practice:

在这种模式下，客户端并不直接创建一个目标对象，而是先调用一个包含了所有必要参数的构造器（或静态工厂）进而得到一个builder对象。接着，客户端调用builder对象提供的类似于setter的方法，并根据喜好开始设置各个想可选参数。最后，客户端通过调用没有参数的build方法生成了目标对象，这个对象通常是不可变的。通常来说，这个builder是它构建的类的一个静态内部类。下面举一个实践中的例子：

```
// Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;
        // Optional parameters - initialized to default values
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        } 
        public Builder calories(int val){ 
            calories = val; return this; 
        }
        public Builder fat(int val){ 
            fat = val; return this; 
        }
        public Builder sodium(int val){ 
            sodium = val; return this; 
        }
        public Builder carbohydrate(int val){ 
            carbohydrate = val; return this; 
        }
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    } 
    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

The \_NutritionFacts \_class is immutable, and all parameter default values are in one place. The builder’s setter methods return the

builder itself so that invocations can be chained, resulting in a fluent API. Here’s how the client code looks:

上述例子中的_NutritionFacts_类是不可变的，所有的默认参数值也放在一个地方。builder的setter方法返回了builder本身，以便这些setter方法的调用可以链接起来，从而代码整体看起来就更流畅些。下面是客户端使用的例子：

```
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();
```

This client code is easy to write and, more importantly, easy to

read. The Builder pattern simulates named optional

parameters as found in Python and Scala.

Validity checks were omitted for brevity. To detect invalid

parameters as soon as possible, check parameter validity in the

builder’s constructor and methods. Check invariants involving

multiple parameters in the constructor invoked by

the build method. To ensure these invariants against attack, do the

checks on object fields after copying parameters from the builder

\(Item 50\). If a check fails, throw an IllegalArgumentException \(Item

72\) whose detail message indicates which parameters are invalid

\(Item 75\).
