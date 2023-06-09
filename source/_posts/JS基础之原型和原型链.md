---
title: JS基础之原型和原型链
date: 2018-05-19 10:44:34
type: "js-base"
tag: js
description:
keywords:
top_img:
mathjax:
katex:
aside:
---

### 解释

#### prototype
我们知道,一切皆是对象,当然函数也不例外.那么既然是个对象,就一定有它的属性,只是很多隐藏的属性我们以前不知道而已.这里我就先说说函数的第一个比较重要的属性--prototype.

当我们创建了一个函数A之后(也就是申明),这个函数A就有了它默认的一个属性prototype,这个属性是内置好了.这时浏览器就会在内存中创建一个"对象B",而前面函数A的prototype的属性的值指向的就是这个"对象B",此时我们就称"对象B"为函数A的原型对象.

#### constructor
他们之间的这种联系并不是简单的prototype的值指向"对象B",其实在"对象B"中也有一个默认的属性constructor,它的值指向了这个函数A!(注:其实任意函数中都有prototype，只不过不是构造函数的时候prototype我们不关注而已)

OK,说到这里,小伙们可能有点绕,那么我直接上图吧!

![img1.png](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfao6lcvu6j30vl0d8weo.jpg)

这是一张简易的原型解析图,就先看最上面俩个框吧.函数A创建完毕后,它的默认属性prototype指向的是浏览器自动生成的对象B,而对象B的内置属性constructor指向的是这个函数A,此时,对象B就是函数A的原型对象!

好的,相信大家在博主生动形象的解析下对原型对象应该有了一个基本概念,那么这个原型对象它有什么用吗?诶,你们还别小看它,它的用处还真挺大的.

#### __proto__
我们知道创建一个对象可以通过构造函数的方式来进行创建.当我们用上面的函数A作为构造函数来创建一个对象A1时,也就是`var A1 = new A();` new一个对象出来.这时,对象A1其实也会有一个默认的属性值`[[proto]]`(由于对象中的`proto`属性是不可访问的,所以用了俩个`[]`来括起来).

当然现在可以用`__proto__`来访问。

就像上面描述的,构造函数A它的默认属性prototype指向的是原型对象B,[[proto]]属性指向的也是原型对象B.  同一个构造函数能用于创建不同的对象,再次利用构造函数A来创建一个对象A2,它的[[proto]]指向的当然也是原型对象B.(现在大家可以回头看看我上面那张图了)

### 例子

```javascript
// 例1:
function Person(name,age){
  this.name = name;
  this.age = age;
}
var person1 = new Person("王先生",22);
var person2 = new Person("张先生",23);
```
person1 和 person2都是通过构造函数Person创建出的对象,所以他俩的proto指向的都是Person的对象原型.

**hasOwnProperty( )方法**
用于判断一个对象中的属性是否来自对象本身,也就是能判断它的来源,它是来自对象本身,还是来自这个对象的[[proto]]属性指向的原型.

若是来自于对象本身,则返回true,	来自于原型和不存在都返回fasle;

将例1稍微改动一下,在Person的原型对象中添加一个eat函数.
```javascript
// 例2:
function Person(name,age){
    this.name = name;
    this.age = age;
}
Person.prototype.eat=function(){
    console.log('a');
}
var person1 = new Person("王先生",22);
var person2 = new Person("张先生",23);
//给person1对象添加属性sex
person1.sex = "男";
console.log(person1.hasOwnProperty('sex'));
console.log(person1.hasOwnProperty('eat'));
=>true
=>false
```
可以看到不管是name,age属性还是eat函数都是在构造函数时就写入了的,也就是都存在于Person的原型对象中,所以第二个console.log输出的就是false,而sex这个属性是我们在创建完对象person1之后添加的属性,所以可以理解为是person1的私有属性,是存在于person1对象本身,所以第一个console.log返回的就是true.

**instanceof操作符和isPrototypeOf()方法**
俩个方法非常相似,都是用于检测一个对象是否来自于一个构造函数
```javascript
//使用instanceof操作符
function A(){ }
var a1 = new A();
console.log(a1 instanceof A);
=>true

function A(){ }
var a1 = new A();
console.log(A.prototype.isPrototypeOf(a1));
=>true
```
isPrototypeOf()函数用于指示对象是否存在于另一个对象的原型链中。如果存在，返回true，否则返回false。

可以简单理解为一个对象是否是通过这个构造函数来创建的.

和instanceof相似,但instanceof是操作符,而isPrototypeOf( )是方法

### 使用组合模型和动态模型

#### 组合模型

简单来说,就是属性在构造函数中创建,而方法在构造函数的原型中创建,如:
```javascript
function Person(name,age){
  this.name = name;						//直接在构造函数中封装属性;
  this.age = age;
}
Person.prototype.eat=function(food){	//在构造函数的原型(Person.prototype)中封装方法;
  alert(this.name+"like eat"+food);
}
Person.prototype.play=function(playName){
  alert(this.name+"like play"+playName);
}

var p1 = new Person("王先生",22);
var p2 = new Person("张先生",23);

p1.eat("拨娜娜");
p2.play("皮革");
```
#### 动态模型

优点:封装性好 
```javascript
function Person(name,age){
  this.name = name;
  this.age = age;
  if(!Person.prototype.eat){							//判断原型中是否有eat函数
    Person.prototype.eat=function(food){				//若没有的话则添加
      console.log(this.name+"like eat"+food)
    }
  }
  if(!Person.prototype.play){
    Person.prototype.play=funciton(playName){
      console.log(this.name+"like play"+playName)
    }
  }
}
//在此可以理解为每调用一次构造函数就执行构造函数,所以每执行一次就会把原先在原型中的函数舍弃,更改为和它一样的函数,则造成了有废弃的函数产生;
var p1 = new Person("王先生",22);
var p2 = new Person("张先生",23);
```
以上俩种模型都有其各自的优点和缺点,那么有没有好点的模型来完善这俩种模型呢,下面来看看这个动态组合模型:

```javascript
function Person(name,age){
  this.name = name;
  this.age = age;
}
Person.prototype = {
  eat:function(food){				
      console.log(this.name+"like eat"+food)
    }
  play:funciton(playName){
      console.log(this.name+"like play"+playName)
    }
}
//在此可以理解为每调用一次构造函数就执行构造函数,所以每执行一次就会把原先在原型中的函数舍弃,更改为和它一样的函数,则造成了有废弃的函数产生;
var p1 = new Person("王先生",22);
var p2 = new Person("张先生",23);
```

#### 终极动态组合模型
```javascript
function Person(ldy){
  this._init(ldy);
}
Person.prototype = {
  _init:function(ldy){
    this.name = ldy.name;
    this.age = ldy.age;
  }
  eat:function(food){				
      console.log(this.name+"like eat"+food)
    }
  play:funciton(playName){
      console.log(this.name+"like play"+playName)
    }
}
//通过向构造函数中传递一个对象opt,这个对象中将要添加的属性添加进去
var p1 = new Person({
  name:"王先生",
  age:22,
})
```
最后这种终极动态组合模型摒弃了以往我们对于构造函数的看法,它在创建对象 p1的时候,传入进去的是一个对象,这样就可以传入不同数量的属性.

并且将要获取的属性全部直接封装到`Person`的原型对象中,这样构造函数`Person`中就只需要调用一下原型对象中的`_init()`函数就可以了(注:`_init`一般用于表示初始化),想要后续添加什么方法直接在`Person`的原型对象中添加.
