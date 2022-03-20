# Java只有值传递

​		

至此之前，Java拥有值传递和引用传递的概念根深蒂固，直到前段时间在地铁上无意看到了Java只有值传递之类的博客，所以决定记录下来。

首先，先介绍一点概念性的东西，既然涉及到了值传递和引用传递，那就先弄懂形参和实参是什么



# 概念

## 形参和实参

- 形参：称为形式参数，术语为<font color="#FF6347">参数</font>，通常用来指该变量作为在函数定义中变量的内容，是一个（未绑定实际值的）变量。

- 实参：称为实际参数，术语为<font color="#FF6347">引数</font>，通常指在函数调用中提供的实际输入，引数可以是一个值或变量，或者是涉及值和变量的更复杂的表达式。

![](https://img.api.liujinshui.com/blog/image-20210918152249219.png)

## 值传递和引用传递

- 值传递：是指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。
- 引用传递：是指在调用函数时将实际参数的地址直接传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。

# 论证

## 传递基本数据类型

### 示例

通过上述概念，我们先通过一个小例子来验证java是否是值传递

```java
public static void main(String[] args) {
        Test test = new Test();
        //在此处我们定义变量
        int num = 10;
        //调用test对象的testValue方法并传递num变量
        test.testValue(num);

        System.out.println("num的值为：" + num);
}

public void  testValue(int i){
        i = 50;
        System.out.println("i的值为：" + i);
}
```

上面代码我们在testValue方法中修改了参数i的值，然后分别打印num和i的值，输出结果为：

```java
i的值为：50
num的值为：10
```

通过这个例子表明，方法testValue并没有改变变量num的值，所以毫无疑问传递基本数据类型可以证明Java是值传递。

### 详解

![](https://img.api.liujinshui.com/blog/image-20210918171027324.png)

大概的流程就是将变量`num`的值`10` 复制一份传入到`test.testValue`的形式参数`i`中，调用testValue方法后对参数`i`进行赋值操作，此时的形参`i`其实就是一个带有值的实参（此处不知这样理解对不对，还请大佬赐教），值为10，而下面又重新给变量`i`进行了赋值操作，此时变量`i`的值为50，所以方法`testValue`对变量`i`进行任何操作也不会影响到`main`方法的变量`num`。<font color="red">**因为java是值传递，所以`num`作为基本数据类型的局部变量是永远无法被其他的外部方法所修改的。**</font>

## 传递对象

### 示例

传递对象

```java
public static void main(String[] args) {
        Test test = new Test();
        User userMain = new User();
    
        //通过set方法写入数据
    	userMain.setUserName("小刘");
    	userMain.setPassWord("123456");
    
        //调用test对象的testValue方法并传递userMain对象
        test.testValue(user);

        System.out.println("main, 对象userMain:" + userMain);
    }

public void  testValue(User user){
        user.setUserName("小水");
        System.out.println("testValue，对象user:" + user);
}
```

结果

```java
testValue，对象user:{userName='小水', passWord='123456'}
main, 对象userMain:{userName='小水', passWord='123456'}
```

运行上述代码后，对象userMain的值被改变了，这不就是引用传递了么。此时就对应那个错误理论：Java的方法中，在传递基本数据类型的时候是值传递，在传递对象的时候是引用传递。

### 详解

将`userMain`对象作为参数传入testValue方法，并不是把`userMain`对象复制一份，而是把实例化后User对象在JVM中堆的地址的地址复制一份。`main`方法中`userMain`本身存储的也是一个地址罢了，所以我们无论在testValue方法中对`user`对象做任何`set`操作，都会改变原有`userMain`所指向的User对象里的内容。（此处可加JVM虚拟机图来表达，有时间加，JVM可参考[深入理解JVM虚拟机](https://blog.csdn.net/csdnliuxin123524/article/details/81303711?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163227548116780269897430%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=163227548116780269897430&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-81303711.first_rank_v2_pc_rank_v29&utm_term=JVM&spm=1018.2226.3001.4187)）

### 例子

你有一把钥匙，当你的朋友想要去你家的时候，如果你直接把你的钥匙给他了，这就是引用传递。这种情况下，如果他对这把钥匙做了什么事情，比如他在钥匙上刻下了自己名字，那么这把钥匙还给你的时候，你自己的钥匙上也会多出他刻的名字。



你有一把钥匙，当你的朋友想要去你家的时候，你复刻了一把新钥匙给他，自己的还在自己手里，这就是值传递。这种情况下，他对这把钥匙做什么都不会影响你手里的这把钥匙。



但是，不管上面哪种情况，你的朋友拿着你给他的钥匙，进到你的家里，把你家的电视砸了。那你说你会不会受到影响？而我们在testValue方法中，改变user对象的userName属性的值的时候，不就是在“砸电视”么。你改变的不是那把钥匙，而是钥匙打开的房子。

# 总结

Java只有值传递，对于对象来讲，这个值实际上是对象的引用。

