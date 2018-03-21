---
title: JavaScript系列之一：创建对象
date: 2017-11-07 13:27:19
categories:
  - 前端技术
tags:
  - JavaScript
---
JavaScript创建对象常见的几种创建对象的方式。

<!-- more -->

### 1. 通过Object构造函数创建对象
```
<script type="text/javascript">
    var person=new Object();
    person.nam="lxy";
    person.age="22";
    person.job="Software Engineer";
    person.sayName= function () {
        alert(this.nam);
    }
    person.sayName();
</script>
```
优点：简单

### 2. 通过字面量创建对象
早期JS开发人员经常使用new Object()创建对象，几年后对象字面量称为创建对象的首选模式。
```
<script type="text/javascript">
    var person = {
        name:"lxy",
        age:22,
        job:"Software Engineer",
        sayName:function(){
            alert(this.name);
        }
    };
    person.sayName();
</script>
```
要注意一点就是每声明一个键值对后面标点是“,”。

这些属性在创建时都带有一些特征值（characteristic），JavaScript通过这些特征值来定义它们的行为。

对象字面量相对于Object构造函数代码量少了一点点。但是这2种方法通过一个接口创建很多对象，会产生大量重复代码。Don't Repeat Yourself！我们需要对重复的代码进行抽象。工厂模式就是在这种情况下出现的。

### 3. 工厂模式
工厂模式是软件工程领域一种广为人知的设计模式，这种模式抽象了创建具体对象的过程。

通过类来创建多个实例必然可以减少代码重复，但是ECMAScript中无法创建类，所以就用函数来封装以特定接口创建对象的细节。

```
<script>
    function createPerson(name, age, job){
        var o=new Object();
        o.name=name;
        o.age=age;
        o.job=job;
        o.sayName=function(){
            alert(this.name);
        }
        return o;
    }
    var lxy=createPerson("lxy", 22, "Software Engineer");
    var strangerA=createPerson("strangerA", 24, "Doctor");
    lxy.sayName();
    strangerA.sayName();
</script>
```
工厂模式减少了重复代码，但是不能够识别对象，所有实例都是object类型的。这时构造函数模式就出现了。

### 4. 构造函数模式
像Object和Array这样的原生构造函数，在运行时会自动出现在执行环境中。我们可以创建自定义构造函数，从而创建特定类型的对象。

```
<script>
    function Person(name ,age,job){
        this.name=name;
        this.age=age;
        this.job=job;
        this.sayName=function(){
            alert(this.name);
        }
    }
    var lxy=new Person("lxy",22,"Software Engineer");
    var strangerA=new Person("strangerA",24,"Doctor");
    lxy.sayName();
    strangerA.sayName();
</script>
```
构造函数中首字母大写，而非构造函数首字母小写作为区别。

通过new操作符来创建Person实例，这样创建的实例都有一个constractor(构造函数)属性，该属性指向Person。

```
alert(lxy.constructor==Person); //true
alert(strangerA.constructor==Person); //true
```
lxy和strangeA是Person的实例，同时也是Object的实例。因为所有的对象都继承自Object。

创建自定义构造函数意味着将来可以将它的实例标识为一种特定的类型；而这正是构造函数胜过工厂模式的地方。

构造函数也是函数，所以语法上可以像普通函数一样去用，但是可以用并不代表应该用，还是以构造函数的方式用更合理。

构造函数的问题是，同一构造函数的不同实例的相同方法是不一样的。

alert(lxy.sayName==strangerA.sayName());//false

这个问题很好理解，因为js中函数就是对象，每定义一个函数，也就是实例化了一个对象。

从代码的角度可能理解的更深刻：
```
this.sayName=function(){alert(this.name)};与
this.sayName=new Function(alert(this.name));是等价的。
```
所以使用构造函数创建对象，每个方法在每个实例上都要重新实现一遍，一是耗资源，二是创建两个或者多个完成同样任务的Function没有必要，三是有this在，没必要在代码执行前就把函数绑定到特定对象上。

所以，有一种方法是说把函数定义转移到构造函数外部，代码如下：
```
<script>
    function Person(name ,age,job){
        this.name=name;
        this.age=age;
        this.job=job;
        this.sayName=sayName;
    }
    function sayName(){
        alert(this.name);
    }
    var lxy=new Person("lxy",22,"Software Engineer");
    var strangerA=new Person("strangerA",24,"Doctor");
    lxy.sayName();
    strangerA.sayName();
</script>
```
把sayName()函数的定义转移到构造函数外部，成为全局的函数，构造函数内部把sayName赋为为全局的sayName。这样sayName是一个指向外部函数的指针，因此lxy和strangeA就共享了 全局的sayName函数。
```
alert(lxy.sayName==strangerA.sayName);//true
```
但是这会有更糟糕的问题：全局作用域的函数只能被某个对象调用，这名不副实啊，会造成对全局环境的污染；更糟糕的是构造函数有多少个方法，就要定义多少个全局函数， 那构造函数就丝毫没有封装性可言了。

但是这样的想法是可贵的，为原型模式做了铺垫，构造函数创建对象问题的解决办法是原型模式。
### 5. 原型模式

原型模式就是把构造函数中方法拿出来的基础上，为了避免对全局环境的污染，再做了一层封装，但是毕竟是一种新的模式，它封装的更彻底，而且也不是把所有的函数都封装，而是恰到好处的把构造函数中公共的方法和属性进行了封装。

```
<script type="text/javascript">
    function Person(){
    
    }
    Person.prototype.name="lxy";
    Person.prototype.age=22;
    Person.prototype.job="Software Engineer";
    Person.prototype.sayName=function(){
        alert(this.name);
    }
     
    var lxy=new Person();
    lxy.sayName();
    var personA=new Person();
    personA.sayName();
    alert(lxy.sayName()==personA.sayName());//true
</script>
```

使用原型的好处是可以让所有的实例共享它所包含的属性和方法。完美的解决了构造函数的问题。因为原型是js中的一个核心内容，其信息量很大，所以另作介绍，有兴趣可看《javascript原型Prototype》。

原型也有它本身的问题，共享的属性值如果是引用类型，一个实例对该属性的修改会影响到其他实例。这正是原型模式很少单独被使用的原因。
```
<script type="text/javascript">
function Person(){

}
Person.prototype.name="lxy";
Person.prototype.age=22;
Person.prototype.job="Software Engineer";
Person.prototype.friends=["firend1","friend2"];
Person.prototype.sayName=function(){
    alert(this.name);
}
     
     var lxy=new Person();
     var personA=new Person();
     alert(lxy.friends);//friend1,friend2
     alert(personA.friends);//friend1,friend2
     alert(lxy.friends==personA.friends);//true
     lxy.friends.push("friend3");
     alert(lxy.friends);//friend1,friend2,friend3
     alert(personA.friends);//friend1,friend2,friend3
     alert(lxy.friends==personA.friends);//true
</script>
```
### 6. 构造函数和原型混合模式
创建自定义类型的最常见方式，就是组合使用构造函数模式和原型模式。构造函数模式用于定义实例属性，而原型模式用于定义共享的方法和属性。结果，每个实例都有一份实例属性的副本，同时又共享着对方法的引用，最大限度的节省了内存。另外，这种混合模式还支持向构造函数传递参数，可谓是集两种模式之长。
```
<script type="text/javascript">
    function Person(name,age,job){
        this.name=name;
        this.age=age;
        this.job=job;
        this.friends=["firend1","friend2"];
    }
    
    Person.prototype={
        constructor : Person,
        sayName : function(){
            alert(this.name);
        }
    }
     
     var lxy=new Person("lxy",22,"Software Engineer");
     var personA=new Person("personA",25,"Doctor");
     alert(lxy.friends); //friend1,friend2
     alert(personA.friends); //friend1,friend2
     alert(lxy.friends==personA.friends); //false
     lxy.friends.push("friend3");
     alert(lxy.friends); //friend1,friend2,friend3
     alert(personA.friends); //friend1,friend2
</script>
```

实例属性在构造函数中定义，而共享属性constructor和共享方法sayName()在原型中定义。而修改一个实例的friends不会影响其他实例的friends，因为它们引用不同数组，根本没关系。

这种构造函数与原型混合模式，是目前使用最广泛、认同度最高的一种创建自定义类型的方法。可以说，这是用来定义引用类型的一种默认模式。其实原型就是为构造函数服务的，配合它来创建对象，想要只通过原型一劳永逸的创建对象是不可取的，因为它只管创建共享的属性和方法，剩下的就交给构造函数来完成。

### 7. 动态原型模式
个人觉得构造函数和原型混合模式已经可以完美的完成任务了。但是动态原型模式的提出是因为混合模式中用了构造函数对象居然还没创建成功，还需要再操作原型，这在其他OO语言开发人员看来很别扭。所以把所有信息都封装到构造函数中，即通过构造函数必要时初始化原型，在构造函数中同时使用了构造函数和原型，这就成了动态原型模式。真正用的时候要通过检查某个应该存在的方法是否有效，来决定是否需要初始化原型。
```
<script type="text/javascript">
    function Person(name,age,job) {
        this.name=name;
        this.age=age;
        this.job=job;
        this.friends=["firend1","friend2"];
        if(typeof this.sayName != "function") {
            alert("初始化原型");//只执行一次
            Person.prototype.sayName=function(){
                alert(this.name);
            }
        }
    }
     
    var lxy=new Person("lxy",22,"Software Engineer");
    lxy.sayName();
    var personA=new Person("personA",25,"Doctor");
    personA.sayName();
</script>
```

使用动态原型时，不能使用对象字面量重写原型。如果在已经创建了实例的情况下重写原型，那么就会切断现有实例与新原型之间的联系。

### 8. 寄生的构造函数模式
在前面几种模式都不适用的情况下，适用寄生（parasitic）构造函数模式。寄生模式其实就是把工厂模式封装在构造函数模式里,即创建一个函数，该函数的作用仅仅是封装创建对象的代码，然后再返回新创建的对象，从表面看，这个函数又很像典型的构造函数。
```
<script type="text/javascript">
    function Person(name,age,job){
        var o=new Object();
        o.name=name;
        o.age=age;
        o.sayName=function(){
            alert(this.name);
        }
        return o;
    }
    var lxy=new Person("lxy",22,"Software Engineer");
    lxy.sayName();
</script>
```

除了适用new操作符使得该函数成为构造函数外，这个模式和工厂模式一模一样。

返回的对象和构造函数或者构造的原型属性之间没有任何关系，所以不能用instanceof，这种方式创建的对象和在构造函数外面创建的对象没什么两样。

### 9. 稳妥的构造函数模式

稳妥的构造函数模式用显式声明的方法来访问属性。

和这种模式相关的有一个概念：稳妥对象。稳妥对象，指没有公共对象，而且其方法也不引用this的对象。

稳妥对象最适合在一些安全的环境中（这些环境中会禁用this和new），或者防止数据被其他应用程序（如Mashup）改动时使用。

稳妥构造函数遵循与寄生构造函数类似的模式，但有两点不同：一是新创建对象的实例方法不引用this，而是不使用new操作符调用构造函数。
```
<script type="text/javascript">
    function Person(name,age,job){
        //创建要返回的对象
        var o=new Object();
        //可以在这里定义私有变量和函数
        
        //添加方法
        o.sayName=function(){
            alert(name);
        }
        return o;
    }
    var lxy=new Person("lxy",22,"Software Engineer");
    lxy.sayName();
    alert(lxy.name);//undefined
    alert(lxy.age);//undefined
</script>
```
以这种模式创建的对象中，lxy是一个稳妥对象，除了使用sayName()方法外，没有其他方法访问name的值，age，job类似。

即使有其他代码会给这个对象添加方法或数据成员，但也不可能有别的方法访问传入构造函数中的原始数据。

与寄生构造函数模式类似，使用稳妥构造函数模式创建的对象与构造函数之间也没有什么关系，因此不能用instanceof操作符。

各种创建方法问题总结：
1. 工厂模式：没法知道一对象的类型。
2. 构造函数：多个实例之间共享方法。
3. 原型：属性值是引用类型时，一个实例对该属性的修改会影响到其他实例。
4. 组合使用构造函数模式和原型模式：【推荐】构造函数模式用于定义实例属性，每个实例都有一份实例属性的副本；而原型模式用于定义共享的方法和属性，每个实例同时又共享着对方法的引用