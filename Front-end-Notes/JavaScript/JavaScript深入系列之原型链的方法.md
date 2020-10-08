# JavaScript深入系列之继承的方法



## 信息卡片

>记录时间：2020-10-07
>
>Tag：JavaScript
>
>更新时间：2020-10-08



## 前言
有半年没更新文章，上半年一直处于焦虑的状态，不怎么想动手提笔，直到现在，才意识到，一直焦虑下去也不是办法的，还不如沉淀自己，提升自己的，即使个人有些特殊情况的，但我相信金子总会闪闪发光的，还是不会闪闪发光的话，但至少我试过～

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/161ac88e79114f099d04faa610d7d793~tplv-k3u1fbpfcp-zoom-1.image)

今天是中秋国庆长假最后一天，抓住尾巴写完一篇文章，继续写上一篇的续集～

> 此文需要准备好电脑，大脑，一杯咖啡

## 1.原型链继承

上一篇文章介绍到过原型链的概念，其实这是一种继承的方法之一，因为在ECMAScript是没有类的概念，然而有些时候自己具有独立的属性或方法，但有些地方跟另一个对象的属性或方法很像的，比如说我们之前新建过一个动物Animal类，然后这个类包含名字，身高，爱吃什么，有多重呢等等？代码如下：

```javascript
function Animal(name,age,height){
	this.name=name;
    this.age=age;
    this.height=height;
}
Animal.prototype.eatFood=function(food){
	console.log(`${this.name}喜欢吃${food}`);
}
Animal.prototype.getWeight=function(weight){
	console.log(`${this.name}体重有${weight}kg`);
}
let buouCat = new Animal('布偶猫',3,10);
buouCat.eatFood('🐟'); //布偶猫喜欢吃🐟
buouCat.getWeight(30); //布偶猫体重有30kg
```

然后有一天新的需求来了，需要对鸟类进行详细的记录呢，是不是会飞的，还要有名字，多大，多高，喜欢吃什么，有多重呢？想看看这个鸟类也是动物类之一的，需要记录的东西，Animal类就有的，所以只要继承就好了，而怎么实现继承的呢？还是利用原型让一个对象继承另一个对象的属性和方法，看👇code:

```JavaScript
//新增的代码
function Bird(isFly){
	this.isFly = isFly;
}
Bird.prototype = new Animal();
Bird.prototype.isOrFly = function(){
	console.log(`${this.name}是${this.isFly ? '会飞' : '不会飞'}的鸟类`);
};
let b1 = new Bird(true);
b1.name = "燕子";
b1.isOrFly(); // 燕子是会飞的鸟类
b1.eatFood('虫子'); //燕子喜欢吃虫子
```

利用Bird的原型来实例化Animal构造函数的，然后Bird的原型指针向Animal，这就扩展了原型搜索的，首先会在Bird搜索该属性，找不到的就沿着Bird的原型往上搜索的，找到的就会返回，找不到的继续往上搜索，直到原型链的末端才停下来呢，这就是原型链的继承方法。当然如果Bird类重写喜欢吃什么样的虫子，就会覆盖Animal类的eatFood方法，看👇code：

```JavaScript
Bird.prototype.eatFood(food1,food2){
	console.log(`${this.name}喜欢吃${food1}，但它一般吃${food2}`);
}
b1.eatFood('大米','虫子'); //燕子喜欢吃大米，但它一般吃虫子。
```

原型链看起来很强大的，但它还是有弱点的，上一篇文章中的“原型”介绍到过的，就是在引用类型方面上还是会影响到所有的实例对象，原型链也是如此的，看👇code：
```javascript
//假如在Animal构造函数上新增一个引用类型的属性
function Animal(name){
	this.name = name
    this.info={
    	isSleep: true,
        isEat: true,
    }
}
Animal.prototype.eatFood=function(food){
	console.log(`${this.name}喜欢吃${food}`);
}
Animal.prototype.getWeight=function(weight){
	console.log(`${this.name}体重有${weight}kg`);
}
function Bird(){
}
Bird.prototype = new Animal();
let b1 = new Bird();
let b2 = new Bird();
//然后我要修改b1的info引用类型
b1.info.isSleep=false;
//然后再看看b2的info有没有变化呢？
console.log(b2.info.isSleep); //false
```

哈？不应该啊，我只是想改下b1这一个的属性而已呢，结果却影响到b2的，那是因为Animal的所有属性是在Bird的prototype上呢，然后Bird实例化b1和b2对象，它们就有同一个prototype属性，要改b1的info，也就相当于改变prototype上的info，然后读取b2的info时，就是从该prototype的info属性（此时这个已经被改变了），所以才会返回false；所以这就是原型链的一个弱点，所以出现借用构造函数

## 2.借用构造函数
说完整点，就是一个子类型构造函数里借父类型的构造函数来调用，因此可以通过call和apply来执行的，这样Bird就拥有Animal的所有属性，这就是借用构造函数的过程，优点是实例化多个对象时，每个实例对象都是独立的，互不影响，看👇code：

```javascript
function Animal(name){
	this.name = name
    this.info={
    	isSleep: true,
        isEat: true,
    }
}
function Bird(){
	Animal.call(this);
}
let b1 = new Bird();
let b2 = new bird();
b1.info.isSleep = false;
b2.info.isSleep = true;//没被影响到的
```

很遗憾的，仅靠借用构造函数的话，就是没办法复用父类型的原型对象，看👇Code：

```javascript
function Animal(name){
	this.name = name
    this.info={
    	isSleep: true,
        isEat: true,
    }
}
Animal.prototype.sayName = function(){
	console.log(`这个动物的名字是${this.name}`);
}
function Bird(name){
	Animal.call(this,name);
}
let b1 = new Bird('麻雀'); 
b1.sayName(); //报错：VM3962:1 Uncaught TypeError: b1.sayName is not a function
```
找不到b1.sayName(), 因为call或者apply只能执行构造函数的，并不包括构造函数的原型对象，所以找不到的，才会报错的，所以这就是缺点，这就出现组合继承的概念，不得不感叹es原来有这么多故事呢～

## 3.组合继承
虽然原型链和借用构造函数都有缺点的，但我们不能因为白板上的一个小黑点就否定所有的，它们还是有自己的优点，原型链优点就是能继承原型属性和方法，而借用构造函数优点就是实例对象能独立继承构造函数的属性，所以把这两个的优点组合在一起，就称为组合继承的，怎么实现，还是用上面的代码来优化下，看👇code：

```javascript
function Animal(name){
	this.name = name
    this.info={
    	isSleep: true,
        isEat: true,
    }
}
Animal.prototype.sayName = function(){
	console.log(`这个动物的名字是${this.name}`);
}
function Bird(name){
	//借用构造函数继承属性
	Animal.call(this,name);
}
//新增代码
Bird.prototype = new Animal();//原型链继承
let b1 = new Bird('麻雀'); 
b1.sayName(); //这个动物的名字是麻雀

let b2 = new Bird('猫头鹰');
b2.info.isSleep = false;
b1.info.isSleep //true;
```

改变info里的isSleep，不影响另一个实例对象的info（借用构造函数的功劳），然后又能打印sayName方法(原型链的功劳）～所以这个成为JavaScript最常用的继承方法，所以说这是最完美的么？并不是呢，它也有不足的，稍后再说这个～

## 4.原型式继承
和上一篇文章中的原型模式类似，利用原型把一个已知对象赋值给未知对象，看👇代码：

```javascript
function object(o){
	function F(){}
    F.prototype = o
    return new F();
}
let animal = {
	name:'🐶',
    info:{
    	isSleep: true,
        isEat: true
   	}
}
let cat = object(animal);
cat.name = '🐱';
cat.name; //🐱
cat.info.isCute = true;
animal.info; //{isSleep: true, isEat: true, isCute: true}
```

要说优点，如果就仅仅要求对象一样的，可以考虑用这个的，但引用类型还是一个缺点的，和原型链缺点一样的，所以谨慎用～

## 5.寄生式继承
这个是在原型式继承上改进的，创建一个封装继承过程的函数，然后返回该对象，用法比较像工厂模式，看👇code：

```javascript
function object(o){
	function F(){}
    F.prototype = o
    return new F();
}
function create(obj){
	let clone = object(obj);
    clone.sayName = function(){
    	cobsole.log(this.name);
    }
    return clone;
}
let animal = {
	name:'🐶',
    info:{
    	isSleep: true,
        isEat: true
   	}
}
let a1 = create(animal);
a1.sayName(); // '🐶'
```

缺点就是不能实现函数的复用～但它的优点还是可以值得借鉴，何出此言呢？继续看👇就知道了～

## 6.寄生组合式继承
嗯，寄生式+组合式来继承的，组合式继承上面讲到过它也有不足之处，不足在哪儿呢？我们先看看code：

```javascript
function Animal(name){
	this.name = name
    this.info={
    	isSleep: true,
        isEat: true,
    }
}
Animal.prototype.sayName = function(){
	console.log(`这个动物的名字是${this.name}`);
}
function Bird(name){
	Animal.call(this,name);//第二次调用Animal构造函数
}
Bird.prototype = new Animal();//第一次调用Animal构造函数
let b1 = new Bird('麻雀'); 
b1.sayName(); //这个动物的名字是麻雀

let b2 = new Bird('猫头鹰');
b2.info.isSleep = false;
b1.info.isSleep //true;
```
其中Bird类两次调用Animal构造函数，第一次是在Bird原型上调用Animal构造函数`Bird.prototype=new Animal()`，Bird.prototype就有了name和info，第二次是调用Bird构造函数时，又会调用Animal构造函数`Animal.call(this,name)`，然后b1有了name和info，屏蔽Bird.prototype上的name和info，看下面的截图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ecb98044c8a746c29bde6780c55026e8~tplv-k3u1fbpfcp-zoom-1.image)

我们本来目的是继承父类的原型对象，并不包括构造函数的所有属性，所以我们需要的无非就是父类的原型的一个副本而已呢，所以这就需要用到寄生式继承的，可以把父类的原型对象看作一个对象，然后赋值给子类的原型

```javascript
function object(o){
	function F(){}
    F.prototype = o
    return new F();
}
function clonePrototype(child,parent){
	let pro = object(parent.prototype); //新建一个对象
    pro.constructor = child; //别忘把construcotr指向子类型，这样instanceof才能正常使用
    child.prototype = pro; //赋值给子类型的原型
}
function Animal(name){
	this.name = name
    this.info={
    	isSleep: true,
        isEat: true,
    }
}
Animal.prototype.sayName = function(){
	console.log(`这个动物的名字是${this.name}`);
}
function Bird(name){
	Animal.call(this,name);
}
clonePrototype(Bird,Animal);
Bird.prototype = new Animal();//第一次调用Animal构造函数
let b1 = new Bird('麻雀'); 
b1.sayName(); //这个动物的名字是麻雀
```

然后我们来看看用寄生组合式继承的打印结果：
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2a66c58e162437bb32c9c8cf5d332f9~tplv-k3u1fbpfcp-zoom-1.image)

可以发现，Bird的原型上没有name和info，这就可以避免重复的，与此同时还能保持原型链的特征，所以这个是最佳的继承方法～

好了，咖啡喝完了，就到此为止～如有错误之处，请多多指出的，笔芯～



