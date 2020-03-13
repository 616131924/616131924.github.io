---
title: Javascript中的原型与原型链
date: 2020-03-13 19:47:49
header_img : https://img-blog.csdnimg.cn/20190311194017886.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjMTg4Njg4NzY4Mzc=,size_16,color_FFFFFF,t_70
categories : JavaScript基础
---
# 浅谈JavaScript中的原型与原型链

### 前言
###### 1.分享这篇文章的目的
原型和原型链其实一直是我的一个知识盲区，一直我都是通过死记硬背的方式来记住他们，但时间过久了，原型和原型链的知识点很快就会变得模糊，因为他们之间的关系是比较复杂的。特别是看到下面这种图的时候。
![image](https://img-blog.csdnimg.cn/20190311194017886.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjMTg4Njg4NzY4Mzc=,size_16,color_FFFFFF,t_70)
如此复杂的关系是很难靠死记硬背记住的，其实我们只需要知道了解原型和原型链的背景，然后再去理清prototype，\_\_proto\_\_，constructor三者各自的含义，自然就能很好的理解JavaScript中原型与原型链到底是个什么东西了
###### 2.为什么要理解原型与原型链
JavaScript是一门面向对象的语言，身为前端工程师，天天都用着JavaScript，但却不知道他到底是怎么面向对象的，这种问题一直都是很多前端工程师的痛点，工作中很少用到，并不意味着我们就能不知道，万一哪天产品提需求让你写一个Vue呢？

### JS中的原型与原型链概念
##### 原型诞生的背景
要讲清楚原型的概念，我们必须了解其背景。JavaScript是一门面向对象的语言，即使我们平时的工作好像不怎么面向对象，但这是不能否认的事实。而设计JavaScript的***Brendan Eich***大神，认为JavaScript只是一个脚本语言并不需要太过庞大的面向对象体系，所以在综合的考虑下，他选择了基于原型的面向对象设计，从一开始原型的概念就已经加入到了JavaScript中。
##### 原型与原型链
###### 1.原型
JavaScript中原型概念与设计模式中的原型模式是一致的，所有的对象都是从另一个对象克隆得来，在创建对象前需要先获取到被创建对象的原型对象，然后进行拷贝形成一个新的对象，而被创建对象中会保存一个指针，该指针指向其原型对象。

###### 1.原型链
在描述原型这个概念的时候，已经说到在对象被创建后会在对象内保存一个原型对象的指针，当生成一个对象B，对象B的原型对象是A，生成一个对象C，对象C的原型对象是对象B,于是就会形成A<-B<-C的单向链结构,而这条链就是原型链，也正是形成了这样的单向链结构，巧妙的利用原型模式在JavaScript中完成了'继承'的实现

### JS中的原型与原型链体现
###### 1.__proto__
![image](https://img-blog.csdnimg.cn/20190311192930650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjMTg4Njg4NzY4Mzc=,size_16,color_FFFFFF,t_70)
__proto__是每个对象都会具备的属性，该属性是一个引用指向当前对象的原型对象，即指向原型prototype，至于为什么prototype前面会有Object.,Foo.,后面的内容会讲到。而__proto__的指向其实就是原型链的指向，当访问图中f1变量中不存在的属性时会沿着__proto__指向访问下一个原型对象，如果直到最后都没有找到匹配的属性，那么就会变成null.xxxx，这个时候就会报错。

在JavaScript的发展历史中，__proto__在一开始并不是ECMAScript规范的内容，而是部分浏览器私自挂载在对象上的一个属性，该属性指向了当前对象的原型对象。而到了ECMAScript2015规范中，__proto__才被纳入规范。

![image](https://7a79-zy-test-1257835692.tcb.qcloud.la/1573549832885_%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_15735493019131.png?sign=ee0b28cf69ad155d88fcd866c8a8a4b5&t=1573550066)

从截图中看出，其实对象中的__proto__属性是被get/set封装过，其实get/set的封装就相当于分别调用Object.getPrototyoeOf(this)/Object.setPrototyoeOf(this,vale) 两个方法来获取当前对象的原型对象

###### 2.prototype
![!image](https://img-blog.csdnimg.cn/20190311193033876.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjMTg4Njg4NzY4Mzc=,size_16,color_FFFFFF,t_70)

 1.1 prototype的真正含义
 
prototype的翻译就是原型的意思，从ES2019规范里prototype被描述为一个给其他对象提供共享属性的对象，这句话表明了prototype本身是一个对象，这个对象的职能是为其他对象提供克隆的模板。在JavaScript中，prototype是作为一个独立的对象存在。

 1.2 函数中的prototype属性
 
在JavaScript中，函数是一个很特别的存在，函数也是对象它具有自己的原型（即__proto__），但函数也能作为某个对象的构造函数而存在。作为构造函数，该函数的作用就是按照指定的原型对象创建一个具体的对象，而指定的原型对象则会保存在函数的prototype属性中，而被创建出来的具体对象中的__proto__会指向该函数的prototype。

###### 3.contructor
![!image](https://img-blog.csdnimg.cn/20190311193745414.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjMTg4Njg4NzY4Mzc=,size_16,color_FFFFFF,t_70)
由图中可以看出constructor属性指向当前对象的构造函数，实例对象中是不存在constructor属性的（手动添加constructor属性除外），但能通过原型链的访问到原型对象上的constructor属性，也就是说constructor属性是存在于prototype中的。

### 原型与原型链的应用
##### 浅析Vue源码内原型与原型链的使用

###### 1.从Vue.extend了解Vue实例与VueComponent实例间的继承
首先，我们经常在代码中操作组件中的this，我们先将this输出来看看。

```javascript
VueComponet{
    $attrs: (...)
    ...
    __proto__: Vue
}

```
如上代码所示，其实每一个组件都是由一个Vue原型对象创建而来，在Vue的说明文档中，Vue.extend的解析为‘使用基础 Vue 构造器，创建一个“子类”’，转成代码的意思是Vue.extend生成一个组件构造函数，如下代码所示。
```javascript
 Vue.extend = function (extendOptions: Object): Function {
    ...
1    const Super = this
    ...
2    const Sub = function VueComponent (options) {
      this._init(options)
    }
3   Sub.prototype = Object.create(Super.prototype)
    ...
4   return Sub
  }
}
```
实现Vue与VueComponet之间的继承分为以下三步：

1.利用Super变量保存当前的调用extend方法的Vue实例对象

2.进行VueComponet构造函数的编写

3.指定构造函数中的prototype属性，即告诉构造函数需要根据指定的prototype进行对象的创建，而prototype指向的是利用Object.create(Vue当前实例对象)创建出来的对象，此时就VueComponent实例对象就能够访问到Vue实例对象上的属性，也就形成了所说的原型链，其中继承已经在这一步完成了。

4.最后一步则将VueComponet的构造函数返回，也就完成了Vue文档中‘创建一个“子类”’的说法

###### 2.this.$emit方法挂载

```javascript
 Vue.prototype.$emit = function (event: string): Component {
    const vm: Component = this
   ...
    return vm
  }
  ```
this.$emit()方法其实并不是VueComponet实例对象上的方法，而是VueComponet继承了Vue对象，继而形成了原型链，当访问到VueComponet.$emit()时会访问Vue.$emit(),而源码中$emit()是通过以上方法挂载到Vue对象上的。

### 拓展
###### 编写一个实现继承的函数
该函数传入一个构造函数a与一个构造函数b和b类中需要的属性，返回一个b类继承a类后新的的构造函数
```javascript
 const inherit =(fatherConstructor,sonConstructor,properties) =>{
   1  let subConstructor = function(...properties){
         fatherConstructor.call(this,...properties)
         sonConstructor.call(this,...properties)
        }
   2  subConstructor.prototype ={
         ...properties,
         constructor:subConstructor
        }
   3  Object.setPrototypeOf(
        subConstructor.prototype,
        fatherConstructor.prototype
     )
     return subConstructor
 }
  ```
步骤分析：

1.创建一个新的构造函数（称为c构造函数），该函数内部调用了父类与子类的构造函数，确保构造函数中初始化的时候与父类和子类一致。

2.新建一个新构造函数的原型对象，里面要保有所有b类的属性和指向c构造函数的constructor属性

3.将c构造函数的原型对象即subConstructor.prototype的指向对象的__proto__指向父级的原型对象即fatherConstructor.prototype


### 小结
1.JavaScript的面向对象是基于原型模式来设计的

2.JavaScript的继承是通过原型链来实现的

3.JavaScript中函数能作为构造函数并用new function()来构建对象。

