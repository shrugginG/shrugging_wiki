# 第四章 对象与类

## 类之间的关系

订单处理系统中的名词：

* 商品（Item）
* 订单（Order）
* 送货地址（Shipping address）
* 付款（Payment）
* 账户（Account）

类之间最常见的关系：

* 依赖（uses-a）

  Order类使用Account类的对象，因为其需要访问Account对象查看信用状态，但是Item类就不依赖于Account类的对象。

  如果一个类使用或者操纵另一个类的对象，那么它就依赖于另一个类。

* 聚合（has-a）

  Order类中包含着Item类对象。

* 继承（is-a）

  RushOrder继承自Order类。

## 使用预定义类

### 对象与对象变量

Java使用构造器构造新的实例，构造器与类名同名。

如果希望构造出的对象可以多次重复使用，可以将其存放在一个对象之中。

![image-20220504174135786](https://images.shrugging.cn/image-20220504174135786.png)

**对象变量并没有实际包含一个对象，它只是引用一个变量。**

```java
Date deadline = new Date()
```

new 操作符的返回值是一个引用，同时将这个引用存储在变量deadline中。

### 类方法的分类

* 访问器方法（accessor method）

  只访问对象而不修改原对象

* 更改器方法（mutator method）

  修改对象的原有状态

## 用户自定义类

一般用户自定义类不包含main()方法，但包含实例字段与实例方法。

构建一个完整的应用程序，会结合使用多个类，其中只有一个类有main()方法。

```java
package cn.shrugging.corejava.volume1.chapter04;

import java.time.LocalDate;

/**
 * @author shrugginG
 * @date 2022/5/4 18:07
 * @project Study4Java
 * @package cn.shrugging.corejava.volume1.chapter04
 */
public class EmployeeTest {
      public static void main(String[] args) {
            Employee[] staff = new Employee[3];

            staff[0] = new Employee("Carl Cracker", 75000, 1987, 12, 15);
            staff[1] = new Employee("Harry Hacker", 50000, 1989, 10, 1);
            staff[2] = new Employee("Tony Tester", 40000, 1990, 3, 15);

            for (Employee e : staff)
                  e.raiseSalary(5);

            for (Employee employee : staff) {
                  System.out.println("name=" + employee.getName() + ",salary=" + employee.getSalary() + "hireDay=" + employee.getHireDay());
            }
      }

}

class Employee {
      private String name;
      private double salary;
      private LocalDate hireDay;

      Employee(String n, double s, int year, int month, int day) {
            name = n;
            salary = s;
            hireDay = LocalDate.of(year, month, day);
      }

      public String getName() {
            return name;
      }

      public double getSalary() {
            return salary;
      }

      public LocalDate getHireDay() {
            return hireDay;
      }

      public void raiseSalary(double byPercent) {
            double raise = salary * byPercent / 100;
            salary += raise;
      }
}
```

示例中包含了两个类，一个是带有public修饰符的EmployeeTest另一个是Employee。

* 源文件名为EmployeeTest.java，文件名必须与public类是同名的
* 在同一个源文件中只能有一个公共类，但可以有任意数目的非公共类
* ![image-20220504183024920](https://images.shrugging.cn/image-20220504183024920.png)
* 许多程序员习惯将每个类放在单独的源文件中

 ### 剖析Employee类

```java
class Employee {
      private String name;
      private double salary;
      private LocalDate hireDay;

      Employee(String n, double s, int year, int month, int day) {
            name = n;
            salary = s;
            hireDay = LocalDate.of(year, month, day);
      }

      public String getName() {
            return name;
      }

      public double getSalary() {
            return salary;
      }

      public LocalDate getHireDay() {
            return hireDay;
      }

      public void raiseSalary(double byPercent) {
            double raise = salary * byPercent / 100;
            salary += raise;
      }
}
```

* 所有方法都标记为public，这意味着任何类任何方法都可以调用这些方法。 
* private确保只有Employee类自身的方法可以访问这些实例字段

### 构造器

构造器总是结合new运算符来调用，不能对一个已经存在的对象调用构造器来达到重新设置实例字段的目的。

* 构造器与类同名
* 每个类至少有一个构造器
* 构造器可以有任意数目的参数
* 构造器没有返回值
* 构造器总是伴随new操作符一起调用

![image-20220504183650924](https://images.shrugging.cn/image-20220504183650924.png)

![image-20220504183656056](https://images.shrugging.cn/image-20220504183656056.png)

### 隐式参数和显式参数

```java
public void raiseSalary(double byPercent) {
            double raise = salary * byPercent / 100;
            salary += raise;
      }


public void raiseSalary(double byPercent) {
				double raise = this.salary * byPercent / 100;
				this.salary += raise;
		}
```

* 只有显式参数会出现在方法的括号中
* 可以使用this指示隐式参数，这是一种代码风格

### 封装的优点

```java
public String getName() {
            return name;
      }

      public double getSalary() {
            return salary;
      }

      public LocalDate getHireDay() {
            return hireDay;
      }
```

以上为Employee类的访问器方法

想要获得或者设置实例字段的值需要：

* 一个私有的数据字段
* 一个公共的字段访问器方法
* 一个公共的字段更改器方法

注意不要编写返回可变对象引用的访问器方法，如果却有需要，需要先对对象进行克隆再返回。

![image-20220504184902977](https://images.shrugging.cn/image-20220504184902977.png)

![image-20220504184911198](https://images.shrugging.cn/image-20220504184911198.png)

### private访问权限

private访问权限的单位为类而非对象，因此不同对象是可以访问同一类不同对象的私有字段的。

### final实例字段

可以将实例字段定义为final

* final实例字段在所有构造器执行完毕后必须被初始化
* 初始化完成后该字段（引用）不可变
* 不存在针对该字段的set方法
* final修饰符对于基本类型以及不可变字段尤其有用（如果一个类中的所有方法都不会改变其对象，就是不可变对象）
* 对象引用不可变不代表引用的对象本身不可变（你只喜欢一个人，但是那个人本身可能会发生改变）

## 静态字段与静态方法

### 静态字段

static字段属于整个类，而不带static的字段则分属于每个实例对象。

### 静态常量

![image-20220504190018608](https://images.shrugging.cn/image-20220504190018608.png)

### 静态方法

* 可以认为不带this隐式参数的方法就是静态方法
* 静态方法无法访问实例字段（因为没有必要，以为它的操作不在实例而在类上）
* 静态方法可以访问静态字段
* 可以使用对象调用静态方法

可以使用静态方法的两种情况：

1. 对象不需要访问对象状态，因为它需要的参数都可以通过显式参数提供
2. 方法只需要访问静态字段

### 工厂方法

工厂方法是静态方法的另一种常见用途

为何要使用工厂方法？

* 无法重命名构造器使其具备意义
* 使用构造器时无法改变构造对象的类型

### main()方法

每个类都可以包含main()方法，用于单元测试

## 方法参数

两种参数传递方法：

* 按值传递
* 按引用传递

Java总是采用按值传递调用，方法得到的是所有参数的一个副本，方法无法修改传递给它的任何参数变量的内容。

![image-20220504191855138](https://images.shrugging.cn/image-20220504191855138.png)

![image-20220504191934584](https://images.shrugging.cn/image-20220504191934584.png)

实现一个改变对象参数状态的方法是完全可以的，方法得到的是对象引用的副本，但是原来的对象引用与副本都引用了同一个对象。

Java中对方法参数可以做什么？

* 不能修改基本数据类型的值
* 可以改变对象参数的状态
* 不能让一个对象参数引用一个新的对象

## 对象构造

### 重载

![image-20220504192632413](https://images.shrugging.cn/image-20220504192632413.png)

![image-20220504192640297](https://images.shrugging.cn/image-20220504192640297.png)

类字段与局部变量的初始化：

* 方法中的局部变量必须被明确的初始化
* 在类中的字段没有被初始化的话会被自动初始化为默认值（0，false或null）

### 无参数的构造器

没有编写构造器的话，会提供一个无参构造器，这个构造器会将所有的实例字段初始化为默认值。

如果至少提供了一个构造器，而没有提供无参构造器，那么调用无参构造器将会出错。

### 显式字段初始化

可以直接在类定义中为任何字段赋值

![image-20220504193247375](https://images.shrugging.cn/image-20220504193247375.png)

![image-20220504193302247](https://images.shrugging.cn/image-20220504193302247.png)

### 参数名

![image-20220504193336482](https://images.shrugging.cn/image-20220504193336482.png)

## 包

### import

### 静态导入

![image-20220504193605638](https://images.shrugging.cn/image-20220504193605638.png)

# 第五章 继承

## 类、超类和子类

```java
public class Manager extends Employee {
      private double bounds;

      public Manager(String name, double salary, int year, int month, int day) {
            super(name, salary, year, month, day);
            this.bounds = 0;
      }

      public double getSalary() {
            double baseSalary = super.getSalary();
            return baseSalary + bounds;
      }

      public void setBounds(double boudns) {
            this.bounds = boudns;
      }

}
```

### 定义子类

最一般的方法应该放入超类中，而更特殊的方法应该放入子类中。

### 覆盖方法

覆盖超类的原有方法时，如何调用原有超类的方法

```java
public double getSalary() {
      double baseSalary = super.getSalary();
      return baseSalary + bounds;
}
```

super和this：

![image-20220505133946943](https://images.shrugging.cn/image-20220505133946943.png)

继承不会删除任何原有字段或方法

### 子类构造器

```java
public Manager(String name, double salary, int year, int month, int day) {
            super(name, salary, year, month, day);
            this.bounds = 0;
      }
```

* Manager类的构造器是无法访问Employee类的私有字段的，必须通过一个构造器来初始化这些私有字段。
* 使用super调用构造器的语句必须是子类构造器的第一条语句

super的两个用处：

1. 调用超类的方法
2. 调用超类的构造器

**何为多态？**

一个对象变量可以指示多种实际类型的现象。

而能够在运行时自动地选择恰当的方法，称为动态绑定。 

### 理解方法的调用

调用x.f(args)

1. 编译器首先查看对象的申明类型和方法名。编译器会一一列类中所有名为f的方法和其超类中名为f的可访问方法，以供编译器作为可候选方法调用。

2. 确定方法调用中的参数类型。

   > 方法的名字和参数构成了方法的签名，如果子类中定义了一个与超类签名相同的方法，那么子类就覆盖了超类的方法。
   >
   > 返回类型不是签名的一部分，不过覆盖一个方法时，需要保证返回类型的兼容性。允许子类将覆盖方法的返回类型改为原返回类型的子类型。

3. 如果是private、static、final方法或者构造器，编译器可以准确地知道所调用的方法，因为它们属于静态绑定，而如果调用的方法依赖于隐式参数的实际类型，需要在运行时动态绑定。

4. 程序运行并采用动态绑定的调用方法时，虚拟机必须调用与x所引用对象实际类型对应的方法。如果子类定义了被引用的方法，就调用这个方法，子类没有定义这个方法的话就去超类中寻找这个方法。

虚拟机预先为每个类计算出了一个方法表

![image-20220505140751393](https://images.shrugging.cn/image-20220505140751393.png)

![image-20220505140813644](https://images.shrugging.cn/image-20220505140813644.png)

### final关键词

* 类：将类声明为final代表这个类不再会被继承
* 方法：将方法声明为final表示这个类无法在子类中被覆盖
* 字段：将字段声明为final表示这个字段在构造对象之后就无法改变它们的值

final类中的所有方法自动成为final，而字段却不是。

### 抽象类

如果一个类中的一个或者多个方法被声明为abstract，那么这个类就必须被声明为abstract。

**抽象类无法被实例化**，但可以定义抽象类对象的变量。

![image-20220505144908177](https://images.shrugging.cn/image-20220505144908177.png)

### 受保护访问

Java中，保护字段只能所有子类和由同一个包中的类访问。

https://blog.csdn.net/monkeyduck/article/details/14107987

![image-20220505145814033](https://images.shrugging.cn/image-20220505145814033.png)

Java中的4个访问控制修饰符

1. private——仅对本类可见
2. public——对外部完全可见
3. protected——对本包和所有子类可见
4. 默认（包访问权限）——对本包可见

## Object

可以使用object类型的变量引用任何类型的对象

```java
Object object = new Employee("Harry Hacker", 35000);
```

![image-20220505162549939](https://images.shrugging.cn/image-20220505162549939.png)

### equals方法

对于一个对象来说，有时候我们认为两个对象相等的标准并不局限于对象的引用相同，对象的某些或全部的实例字段相同时我们也认为其实相同的。

```java
@Override
public boolean equals(Object o) {
      if (this == o) return true;
      if (o == null || getClass() != o.getClass()) return false;
      Employee employee = (Employee) o;
      return Double.compare(employee.salary, salary) == 0 && Objects.equals(name, employee.name) && Objects.equals(hireDay, employee.hireDay);
}
```

```java
public boolean equals(Object otherObject) {
      // 是否为相同的引用
      if (this == otherObject) return true;

      // otherObject引用是否为空
      if (otherObject == null) return false;

      // 是否为相同类型的引用
      if (getClass() != otherObject.getClass()) return false;

      Employee other = (Employee) otherObject;
      return Objects.equals(this.name, other.name)
            && this.salary == other.salary
            && Objects.equals(this.hireDay, other.hireDay)

}
```

编写一个完美的equals方法：

1. 显示参数命名为otherObject，稍后强制转换为一个名为other的变量

2. 检测this与otherObject是否相同

   ```java
   if (this == otherObject) return true;
   ```

3. 检测otherObject是否为null，是则返回false。

4. 比较this与 otherObject类，如果equals的语义可以在子类中改变，就使用getClass检测：

   ```java
   if (getClass() != otherObject.getClass()) return false;
   ```

   如果所有的子类都具有相等性语义，可以使用instanceof检测：

   ```java
   if (!(otherObject instanceof className)) return false;
   ```

5. 将otherObject强制转换为相应类型的变量：

   ```java
   Employee other = (Employee) otherObject;
   ```

6. 根据相等性概念的要求比较字段

如果要在子类中重新定义equals，就要在其中包含一个super.equals(other)的调用。

![image-20220505165521735](https://images.shrugging.cn/image-20220505165521735.png)

![image-20220505170035882](https://images.shrugging.cn/image-20220505170035882.png)

![image-20220505170100508](https://images.shrugging.cn/image-20220505170100508.png)

![image-20220505170111334](https://images.shrugging.cn/image-20220505170111334.png)

### hashCode

* 默认Object中的hashCode是通过对象的存储地址得出的。
* String类中的hashCode是根据内容生成的。
* 如果重新定义了equals方法，就必须重新定义hashCode方法。

![image-20220505170729985](https://images.shrugging.cn/image-20220505170729985.png)

![image-20220505170808450](https://images.shrugging.cn/image-20220505170808450.png)

### toString()

* 大多数对象的toString()方法会返回类的名字加一对括号括起来的字段值。

  ```java
  @Override
  public String toString() {
        return "Employee{" +
              "name='" + name + '\'' +
              ", salary=" + salary +
              ", hireDay=" + hireDay +
              '}';
  }
  ```

* 也可以使用getClass实现

  ```java
  public String toString() {
        return getClass().getName() +
              "{name='" + name + '\'' +
              ", salary=" + salary +
              ", hireDay=" + hireDay +
              '}';
  }
  ```

  这样可以在子类中直接调用

  ![image-20220505171423096](https://images.shrugging.cn/image-20220505171423096.png)

* 只要一个对象与一个字符串通过+连接就会自动调用对象的toString()方法
* print(x)，也会自动调用x的toString()方法
* Object的默认toString()方法，返回的是类名和散列值。
* ![image-20220505171636952](https://images.shrugging.cn/image-20220505171636952.png)

![image-20220505171718578](https://images.shrugging.cn/image-20220505171718578.png)

## 泛型数组列表

### 声明数组列表

![image-20220505201034816](https://images.shrugging.cn/image-20220505201034816.png)

### 相关操作

![image-20220505201241698](https://images.shrugging.cn/image-20220505201241698.png)

![image-20220505201249356](https://images.shrugging.cn/image-20220505201249356.png)

![image-20220505201433849](https://images.shrugging.cn/image-20220505201433849.png)

### 对象包装器与自动装箱

* 所有的基本数据类型都有一个与之对应的类，这些类被称为包装器（wrapper）
* 包装器都是final，一旦构造了包装器就不允许改变包装器中的值，且不能派生他们的子类。

ArryList尖括号中的类型参数不允许是基本数据类型，因此就需要写入基本数据类型对应的包装类。

```java
ArrayList<Integer> list = new ArrayList<Integer>()
```

![image-20220505202425098](https://images.shrugging.cn/image-20220505202425098.png)

自动装箱：

![image-20220505202458003](https://images.shrugging.cn/image-20220505202458003.png)

==应用于包装器对象还会检测对象是否有相同的内存位置

在比较两个包装器对象时应该调用equals方法

![image-20220505202920489](https://images.shrugging.cn/image-20220505202920489.png)

![image-20220505202932222](https://images.shrugging.cn/image-20220505202932222.png)

### 可变参数

```java
public class VarArgs {
      public static void main(String[] args) {
            double max = VarArgs.max(3.1, 40.4, -5);
            System.out.println(max);
      }

      public static double max(double... values) {
            double largest = Double.NEGATIVE_INFINITY;
            for (double value : values) {
                  if (value > largest) largest = value;
            }
            return largest;
      }

}
```

### 枚举类



* 枚举类的构造函数总是私有的
* 所有的枚举类都默认继承了Enum类，其中的toString方法会返回枚举常量名
* toString的逆方法是静态方法valueOf
* values方法将返回一个包含全部枚举值的数组
* ordinal方法返回enum声明中枚举常量的位置



# 第六章 接口、lambda表达式与内部类

## 接口

Java中接口并不是类，而是希望符合这个接口的类的一组需求

* 接口中所有的方法都自动为public，因此不必提供关键词public，**不过在实现接口中的方法时必须标注为public**
* 接口可以定义常量，但是接口绝不会有实例字段
* Java8之前接口绝不会实现方法，Java8之后接口可以提供简单的方法，而绝不会引用实例字段。
* 提供实例字段和方法实现的任务应该由实现接口的类完成，因此可以将接口看作没有实例字段的抽象类。

### 接口的属性

* 接口中不可以包含实例字段却可以包含常量

* 方法都被自动设为public，而常量都被自动设为public static final

* 接口方法与常量的访问属性声明都可以被省略

* 每个类只能有一个超类，却可以实现多个接口

  ```java
  class Employee implements Cloneable,Comparable
  ```

### 接口与抽象类

为什么不直接使用抽象类而要设计接口？

* Java规定一个类只能继承一个超类，但可以实现多个接口
* 接口可以提供多重继承的大多数好处，还能避免多重继承带来的复杂性和低效性。

### 静态和私有方法

* Java8之后允许在接口中实现静态方法
* 目前为止，通用的做法是将静态方法放入伴随类中（标准库中成对出现的Collection/Collections）

### 默认方法

可以为接口方法提供一个默认实现，用default修饰符标记一个这样的方法

![image-20220506135422576](https://images.shrugging.cn/image-20220506135422576.png)

![image-20220506135439963](https://images.shrugging.cn/image-20220506135439963.png)

接口演化

使用默认方法可以实现对之前版本的兼容

### 解决默认方法冲突

如果一个接口提供了一个默认方法，而又在超类或另一个接口中定义了同样的方法应该如何解决？

1. 超类优先

   如果超类中定义了，那么同名且有相同参数的默认方法会被忽略。

2. 接口冲突

   如果两个接口提供了相同签名的方法，用户就必须覆盖实现这个方法以解决冲突。

“类优先”策略：

如果一个超类与接口中有实现了相同的方法，接口中的默认方法会被忽略。

### Comparator接口

默认String中对字符串的比较是按照字典顺序进行的，如果我们想按照字符串长度进行排序该如何实现：

Arrays.sort()的一个版本可以接受两个参数，一个参数是一个数组和一个比较器，比较器就是实现了comparator接口类的实例。

```java
public class TestComparator {
      public static void main(String[] args) {
            String[] firends = {"Peter", "Paul", "Mary"};
            Arrays.sort(firends, new LengthComparator());
            for (String firend : firends) {
                  System.out.println(firend);
            }
      }
}

class LengthComparator implements Comparator<String> {
      @Override
      public int compare(String o1, String o2) {
            return o1.length() - o2.length();
      }
}
```

### 对象克隆

拷贝和克隆的区别：

![image-20220506164257877](https://images.shrugging.cn/image-20220506164257877.png)

浅拷贝：

![image-20220506164315718](https://images.shrugging.cn/image-20220506164315718.png)

对于每一个类的拷贝，我们需要考虑：

1. object中默认的clone方法是否满足需求。

   默认的clone方法都是浅拷贝，如果被克隆的对象中存在可变的子对象，那么拷贝完成后依然引用的是同一份子对象。

2. 是否可以在可变的子对象上调用clone来完善默认的clone方法；

3. 是否不该使用clone。

第三项为可选项，如果选择第一项或者第二项，类必须：

1. 实现Cloneable接口；
2. 重新定义clone方法，并指定public访问修饰符。

![image-20220506170127716](https://images.shrugging.cn/image-20220506170127716.png)

![image-20220506170439777](https://images.shrugging.cn/image-20220506170439777.png)

## 内部类

内部类的作用：

1. 内部类可以对同一个包中的其他类隐藏
2. 内部类方法可以访问定义这个类的作用域中的数据，包括原本私有的数据

定义内部类并不代表每个外部内实例都有一个内部类实例，内部类实例。

