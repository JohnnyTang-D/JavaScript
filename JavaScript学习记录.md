# JavaScript学习记录

### 一、高级语言分类

高级语言分为解释型语言和编译型语言,其实现代的高级语言通常都是编译+解释相结合,Java python等都需要先编译再解释,这里只是强行分为两类,
进行总结.

1. 编译型语言有C C++ Go等.它们需要编译器编译成机器可以调用的程序,如exe dll等类型,所以程序执行效率极高.
但是因为代码需要经过编译才能运行,所以移植性差,只能在兼容的操作系统上运行.
通常使用场景是开发操作系统 大型应用程序 图像处理 数据库开发
2. 解释型语言有Java Python等.它们不需要编译器,需要解释器.执行到每条语句的时候去解释一下,重复执行就会重复解释,所以运行效率极低.
代码不需要经过编译可以在不同操作系统上运行,只需要一个解释器即可,所以可移植性高.但是因为解释器也占用系统资源,所以运行速度慢.

JavaScript特殊的编译+解释语言.它不是提前编译,而是需要一个专门引擎(目前主流v8)在运行时候编译.通常编译过程会附加很多性能优化.
编译过程如下

- 词法分析/分词

    将字符流分析成记号流.

- 语法分析/解析

    将记号流分析成AST树.

- 代码生成

    根据AST树生成相关代码,会先处理声明变量再定义函数

### 二、作用域

在代码生成阶段会预先处理变量跟函数,同时会生成作用域机制,而JavaScript作用域是在定义时候决定的,代码写在哪里,作用域在哪里,称之为词法作用域.还有一种动态作用域,是在执行的时候确认作用域,通过查找调用栈,
从而找到各个函数的作用域,也就是函数在哪里调用的.在JavaScript中,this很像动态作用域.

比如`var a = 2`这一句代码,在遇到var a会询问作用域是否存在`a`这个名称的变量,有则会忽略,没有则会在当前作用声明.`a = 2`则会进行赋值操作,会询问作用域,
取到了"a"这个变量之后,会进行赋值2;

而以上询问查询过程包括LHS和RHS两种方式,LHS会寻找对变量进行赋值操作的容器也就是内存空间,RHS则是找到该变量的容器内存空间的值.比如 `var a = 2`则是一个LHS操作,
因为会查找`a`这个变量的容器进行赋值.`var b = 2; var a = b`首先会有一个`b = 2`的LHS,`a = b`则首先是LHS`a`这个变量的容器,
随后进行RHS`b`这个变量的取值,最后进行赋值.
```javascript
function foo(a){
  console.log(a); 
}

foo(2);
```
特别注意:在这段代码中,隐含一个`a = 2`的LHS操作.即函数形参传递.

```javascript
var a = 2;
function foo(){
  var a = 3;
  console.log(a);  // 3
}
foo()
console.log(a); // 2
```
此段代码`foo`在执行的时候**,**会首先查找当前foo定义的作用域中,所以会输出一个`3`.`console.log`会在当前定义域中查找,此时会查找到`2`.

JavaScript的作用域是在代码定义时候确定的.一个函数内部是一个作用域称为函数作用域,{}内部是一个作用域称为块作用域.作用域可以嵌套.
也就是说在进行LHS和RHS查询的时候,会根据代码定义时候的作用域进行查找相应的变量,就好像在一栋楼里面找人,通常你从一楼找起,如果没有就会往上一层进行寻找.
作用域嵌套查询也是如此.

### 三、提升

变量声明的时候,通常都用var let const修饰符,var与let主要差别是是否会提升的差别,提升差别所带来的差距就是let会暂时性死区,也就是let声明在后,使用不能在前.
但是var可以先使用,后声明.let且无法重复声明.const拥有let的性质,但是增加了一条无法被修改.

提升的意思是,考虑如下代码

```javascript
a = 2;
var a;
console.log(a)
```

```javascript
console.log(a)
var a = 2;
```

会输出什么,答案是2 undefined;这是因为`var a`会提升到当前作用域的顶部.为什么如此?是因为JavaScript在词法分析的阶段会将`var a = 2`这段代码认为是
`var a`跟`a = 2`两行代码.所以在分析阶段就会将一个变量声明提升到一个作用域的顶部.函数亦是如此,函数提升会首先发生,变量其次.因为在JavaScript中,函数是
一等公民.注意`var a = function var(){}`此段代码是函数表达式,而不是函数声明,所以不会提升函数.想在之前调用`a`会报typeError的错误,因为此时a只是
一个变量,而不是进行了赋值之后的函数.函数声明如果有同名,后面的函数会替换前面的函数.

### 四、闭包

闭包的定义是函数有一个内部函数，而这个内部函数如果传递到不属于其自身作用域的地方，这个地方就会一直引用内部函数所在函数的整个作用域，从而形成不被垃圾回收的闭包。 
- 例子1
```javascript
function handle(){
  var a = 2;
  function setOption(){
    console.log(a);
  }
  start(setOption)
}

function start(fn){
  fn()
}

```
在handle这个函数内部,会将setOption传递给start这个函数.此时就会发生start这个函数作用域拿到了handle这个函数的整个作用域,从而形成了闭包.

- 例子2
```javascript
function wait(mes){
  setTimeout(function timer(){
    console.log(mes)
  },1000)
}
wait("测试")
```
在wait函数中,setTimeout参数传递了一个函数,而此函数属于wait作用域.setTimeout属于其他作用域,所以setTimeout拿到了wait作用域,形成了一个闭包.

### 五、this

this的设置是为了解决`在代码越来越多,显示传递上下文对象会让代码变得越来越混乱,使用this,则无需显式的传递上下文对象`.
```javascript
function add(a,b,c){
  return a+b+c;
}
add(1,2,3)
```
如上代码,在代码越来越多的情况下,显式传入的上下文对象越来越多,代码就会越来越混乱.
```javascript
function add(){
  return this.a + this.b + this.c;
}
const num = {
  a:1,
  b:2,
  c:3
}
add.bind(num)()
```
如上代码,在代码越来越多的情况下,不用再显式传递上下文对象.只需要设置this指向即可.

在知道this解决的痛点之后,我们探求下this的指向问题.

- 误解1.this指向自身
```javascript
function foo(){
  console.log("foo:" + num);
  this.count++;
}
foo.count = 0;
var i;
for(i = 0; i < 10; i++){
  if(i > 5){
    foo(i);
  }
}
console.log(foo.count) // 0
```
显然this指向并不是指向自身.如果指向函数自身,就会count不断++,最后输出应该是个4.当然这个问题也可以解决,就是用词法作用域解决.操作一个公共对象或者将this
明确为哪个函数.

- 误解2.this指向函数作用域
```javascript
function foo(){
  var a = 2;
  this.bar();
}
function bar(){
  console.log(this.a);
}
foo(); // ReferenceError:a is not defind

```
显示this并不是指向函数作用域

所以综上,this指向是函数调用方式决定的.跟函数的声明位置没有任何关系.当函数被调用时,会创建一个执行上下文,上下文会记录函数在哪里调用的,函数的调用方式,
传入的参数等信息.

调用位置规则总结:

1. 默认绑定

独立函数调用或者说在无法应用其他规则的时候默认规则.

```javascript
function foo(){
  console.log(a);
}
var a = 2;
foo() //2
```
`foo()`在调用的时候,应用了默认规则,因为这里是直接使用没有任何修饰函数的东西.而默认规则this指向全局对象window或者global,如果是严格模式,则this是
undefined.ps:如果foo不是默认规则,则this还是window或者global

2. 隐式绑定

调用位置是否有上下文对象或者说是否被某个对象拥有或者包含
```javascript
function foo(){
  console.log(this.a);
}
var obj = {
  a:2,
  foo:foo // 这里只是obj获得了foo这个函数的引用地址,而不是真实拥有这个函数
}
obj.foo();
```
`obj.foo()`这里调用会使用obj这个上下文对象,因此会应用隐式绑定,将obj这个上下文对象绑定到this身上.如果发生`obj1.obj2.foo()`这种链式调用,以最后一次
出现的上下文对象为准.

问题:丢失this
```javascript
function foo(){
  console.log(this.a);
}
var obj = {
  a:2,
  foo:foo
}
var bar= obj.foo
var a = 3;
bar();// 3
```
这里将`obj.foo`这个函数引用赋值给了`bar`,执行`bar()`.但是如同上文所说,`obj`只是获得了`foo`函数的引用地址,而不是真实拥有,所以在执行`bar`时候,
没有上下文对象,而失去了this指向,应用了默认规则,指向了全局对象window或者global.

3. 显式绑定

```javascript
function foo(){
  console.log(this.a);
}
var obj = {
  a:2
}
foo.call(obj)
```
通过使用`call` `apply` `bind`将this强制绑定到某个上下文对象上.其中`bind`是将this改变,然后返回一个新函数.如果你在这些硬绑定的时候传入null,则
会应用默认绑定.

4. new绑定

使用new操作符去调用一个函数,首先会创建一个全新的对象,这个新对象会连接到Prototype上,将这个新对象绑定到函数的this上,如果函数没有返回其他对象,new会自动返回
这个新对象.

4条规则中,优先级最高的是new绑定,其次是显式绑定,再就是隐式绑定,最后是默认绑定.

5. 箭头函数

箭头函数不使用上述4条规则,而是根据外层作用域来决定this.即箭头函数的this是词法作用域决定的,不是运行时调用位置决定.统一了JavaScript词法作用域规则.
箭头函数的this是无法被修改的.最常用的场景是事件处理器或者定时器.

### 六、对象

1. JavaScript有6种主要数据类型:`string` `number` `null` `undefined` `boolean` `object`.其中 `function` `array`属于`object`.
2. JavaScript有内置对象:`String` `Number` `Boolean` `Function` `Array` `Date` `RegExp` `Error`
3. 对象属性可以通过defineProperty去设置属性描述符.属性描述符包括:writable(是否可以修改属性的值)、configurable(是否可以配置)、enumerable(属性是否会出现在对象的属性枚举中)
4. 对象的属性访问是通过内置的`get` `set`内置函数来决定的.

### 七、类

在面向对象设计规则中,类通常是数据+行为的结构体.通过类生成实体,实体使用类的行为去处理数据.类有继承跟多态的特点.

1. 在使用类生成实例的时候,通常会使用类的构造函数初始化实例的状态.通常构造函数名字跟类名是一致的.
2. 继承,可以通过一个类继承另外一个类,实现共性的逻辑抽取.继承的类拥有被继承的类的行为跟数据.
3. 多态,父类的行为可以被子类重写,也可以直接调用父类的行为.

在JavaScript中,只有对象,并不存在可以被实例化的类.一个对象并不会被复制到其他对象,它们会被关联起来,也就是多个引用一份副本.JavaScript用了一种混入的模式模拟了类.

1. 显式混入.通过使用mixin方法将一个对象的属性复制到另外一个对象.实现类的继承.

   ```javascript
   function mixin(sourceObj,targetObj){
       for(var key in sourceObj){
           if(!(key in targetObj)){
               targetObj[key] = sourceObj[key]
           }
       }
   }
   ps:这里的mixin实现的只是浅拷贝,两个对象有可能复用同一份引用.
   ```

2. 隐式混入

   就是使用this去调用方法,在子类中通过使用call apply去改变调用函数的thisi,修改为子类的指向,从而实现调用父类的函数,函数中使用thisi指向了子类.

### 八、原型

prototype是对象的一个特殊属性,是对于其他对象的引用.比如`myObject.a`会触发get操作,去取值,但是这个对象中并没有`a`这个属性,这时候就会去寻找prototype上有无`a`这个属性.会一层一层的往上找,俗称原型链.原型链的终点是Object.prototype是一个内置对象.

```javascript
function Foo(name){
    this.name = name;
}
Foo.prototype.myName = function(){
    return this.name;
}
function Bar(name,label){
    Foo.call(this,name);
    this.label = label;
}
Bar.prototype = Object.create(Foo.prototype);
Bar.prototype.myLabel = function(){
    return this.label;
}

var a = new Bar("a","obj a")

a.myName(); // a
a.myLabel(); // obj a
```

`Bar.prototype = Object.create(Foo.prototype)`中,`Object.create()`会创建跟传入参数一样的新对象返回,所以这一行代码生成了一个Foo.prototype的新对象并赋值给了Bar的原型对象Bar.prototype.

当 `var a = new Bar("a","obj a")`执行的时候,会生成一个新对象,原型指向Bar.prototype,在执行`Bar("a","obj a")`的时候,`Foo.call(this,name)`会将新对象强制绑定到Foo的this上,从而使得新对象上生成了一个name属性,在`a.myName`在`a`这个对象找不到,会去`Bar.prototype`中寻找即`Foo.prototype`中,找到myName之后执行,this为新对象,在新对象上找到`name`这个属性,从而输出 "a".

```javascript
function Foo(name){
    this.name = name;
}
Foo.prototype.myName = function(){
    return this.name;
}
function Bar(name,label){
    Foo(name);
    this.label = label;
}
Bar.prototype = Object.create(Foo.prototype);// 这里会抛弃旧的对象,创建一个新对象.而使用ES6提供的api可以直接修改旧对象.Object.setPrototypeOf
Bar.prototype.myLabel = function(){
    return this.label;
}

var a = new Bar("a","obj a")

a.myName(); //undefined
a.myLabel(); // obj a
window.name //a
```

修改一下代码,不使用call修改this,这时候`Foo`在执行的时候,this应用了默认规则,而默认规则是window,这时候name属性在window上.

### 九、面向委托编程

通过以上几个章节的总结明白了JavaScript是没有类这个概念的.所以我们通过转变思维,使用委托这个概念去描述"类"这个关系,就会发现在JavaScript中事情变得简单了.如下:

```javascript
const Task = {
    setId:function(ID){
        this.id = ID;
    },
    outputId:function(){
        console.log(this.id)
    }
};
const XYZ = Object.create(Task);
XYZ.prepareTask = function(ID,label){
    this.setID(ID);
    this.label = label;
}
XYZ.outputTaskDetails = function(){
    this.outputID();
    console.log(this.label);
}
```

在委托模式中,通过创建`Task`跟`XYZ`两个对象,`XYZ`的原型委托给`Task`,从而轻松实现"类"这一概念,而代码也更符合JavaScript的特点以及优点.

委托行为意味着某些对象(XYZ)在找不到属性或者方法引用的时候会把这个请求委托给另外一个对象(Task)
