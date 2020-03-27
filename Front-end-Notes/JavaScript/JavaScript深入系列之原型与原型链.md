# JavaScript深入系列之原型与原型链



## 前言

每次遇到新的概念，我们可能会想到三个问题，它是什么呢（what）？为什么会出现呢（why）？又能解决什么呢（how）？这次我们来看看原型与原型链这个概念，要解决这个3W（what, why, how），首先得从js的面向对象说起...



## 面向对象

什么是面向对象？应该不需要我多说呢？任何编程语言都有面向对象，然后它们都有类的概念（class），但在ECMAScript（以下用es表示）中却没有，除了es6之外（其实上是语法糖，这样是为了更像对象编程的语法而已），但不代表因为es6有了类，我们就可以忽略原型与原型链的概念，因为es6是在es5的基础上优化来的，



那么问题来了，既然没有类的概念，那es又是如何实现的呢？起初使用工厂模式实现，这个又是什么玩意呢？

## 工厂模式

正如字面的意思，就拿制造娃娃（Doll）的工厂来说吧，工厂要制造一个娃娃的过程：首先制造娃娃的对象（new Object()），然后赋给娃娃的属性，比如颜色，多高，多重，最后就制造出来，代码如下：

```javascript
function createDoll(color,height,weight){ //制造工厂模式的函数
  let o = new Object();
  o.color = color;
  o.height = height;
  o.weight = weight;
  o.sayColor = function(){
    alert(o.color);
  }
  return o;
}
//为了显示是工厂模式，变量名后面加F(工厂：Factory)
let dollF1 = createDoll('orange',20,10);//要制造一个娃娃，就调用上面的函数
let dollF2 = createDoll('red',30,10);//同上
```

把一个对象的所有属性和方法（所谓本身的信息）封装在一个工厂模式的函数，这样就能制造多个对象

* 好处：

  减少大量重复的代码

* 缺点：

无法识别对象的类型问题，换句话说，就是无法判断一个对象来自于哪个函数呢？这个稍后再说，正是因为这个缺点，所以才会出现构造函数模式～

## 构造函数模式

什么是构造函数呢？代码写法其实和普通函数差不多的，但只要被new调用就称为构造函数，代码如下：

```javascript
function Doll(color,height,weight){//本来是普通函数
  this.color = color;
  this.height = height;
  this.weight = weight;
  this.sayColor = function(){
    alert(this.color);
  }
}
//为了表示是构造函数模式，变量名后面加C
let dollC1 = new Doll('orange',20,10);//用new调用了，上面的函数就称为构造函数
let dollC2 = new Doll('red',30,10);
```

为了更好的识别构造函数，函数名应该以一个大写字母开头，比如上面的大写字母D，然后与工厂模式相比，去掉创建对象（ let o = new Object();），直接把属性和方法赋给this对象，去掉return返回语句。

这个有啥优点，之前提到过工厂模式无法识别对象的类型问题，我们先看看调用构造函数和调用工厂模式的打印结果，

> 我们不光要看文章，还要动手打code，这样理解容易多了～


#### 调用工厂模式的结果如下：

![调用工厂函数的结果](https://user-gold-cdn.xitu.io/2020/3/26/17114c496073b115?w=1018&h=746&f=png&s=170612)


#### 调用构造函数的结果如下：


![调用构造函数的结果](https://user-gold-cdn.xitu.io/2020/3/26/17114c4de0045265?w=1018&h=1072&f=png&s=217606)

我们可以发现，调用构造函数比工厂模式多了一个constructor属性，这个属性指向Doll，为了证明这两个是相等，我们可以打印出看看：
```
dollC1.constructor === Doll; //true
//然后我们再来看看工厂模式的又是如何呢
dollF1.constructor === createDoll; //false，因为constructor属性找不到creteDoll，所以无法判断来自于createDoll函数，这就是工厂模式的缺点
```

> 为什么非要判断对象类型，别急，这个要说的原因很长，要一个一个解释，
> 就好比说吃美食，要慢慢咬嚼，才能知道美食是什么味道，相反吃的太快，就说不出味道的，也就没办法向别人（面试官或者写代码）解释这个本质～


这个constructor属性可以用来标识对象类型，但小红书推荐用instanceof来判断，这个是表示可以是Object的实例，也可以是构造函数的实例，看看上面调用工厂模式和构造函数的图片，工厂模式中的dollF1__proto__下面有一个constructor，指向Object；而构造函数中的dollC1__proto__下面有两个constructor，分别指向Object和Doll，然后我们运用instanceof来判断：

```
//构造函数模式
dollC1 instanceof Doll; //true
dollC1 instanceof Object; //true
//工厂模式
dollF1 instanceof createDoll; //false
dollF1 instanceof Object; //false
```

所以这就是工厂模式和构造函数模式的优劣比，至于为什么会是Object的实例，这个稍后再说～

然而不要太天真了，任何事物绝对没有完美的，构造函数也是如此的，不然我就不会讲原型与原型链，那么构造函数有什么缺点呢？我们先看构造函数中的sayColor方法，这个方法在多个实例对象上应该是一样，比如dollC1和dollC2：

```
dollC1.sayColor === dollC2.sayColor; //false
```
结果打印出false，哈？为什么呢？调用构造函数后，会生成不同作用域的多个实例，相当于开辟多个实例的内存，拥有自己的属性可以理解的，但拥有自己的方法这就没必要，就好比说两个人要去旅行，需要准备东西，由于卫生问题，带上自己的毛巾（属性）可以理解的，然后你和朋友都要带上沐浴液，带的越多，提的就越重，换个角度想，如果不带自己的沐浴液，就借朋友的沐浴液来用，这样可以减少自己的容量。在构造函数也是一样的道理，带自己的属性和方法越多，开辟的内存就越大，所以出现原模型型这个概念（终于到这一步了🤪）


## 原型模型（也可以叫原型）
什么是原型模式呢？原型模式也可以这么叫共享模式（个人的理解哈），这几年不是很流行共享嘛？共享单车，共享充电宝，共享汽车等等，一个单车可以被所有人共用，相当于一个原型模式可以被所有的实例共用，区别在于单车要花钱哈～

之前讲到的构造函数的属性和方法，这次在原型模式中，把属性和方法移到构造函数的原型对象，用prototype实现：
```
function Doll(){};
Doll.prototype.color='orange';
Doll.prototype.height=12;
Doll.prototype.weight=6;
Doll.prototype.sayColor=function(){
    cobsole.log(this.color);
}
//为了表示是原型模式，变量名后面加P
let dollP1 = new Doll();
dollP1.sayColor();  //orange;
let dollP2 = new Doll();
```

这就是原型模式，之前说的构造函数有个缺点的，多个实例对象的方法不一致，这次我们来看看多个实例对象的方法在原型模式又是如何呢？

```
dollP1.sayColor === dollP2.sayColor; //true
```

一样的耶

## 原型链

