# 面向对象编程
## 一、JS的解析和执行过程
  程序执行前，首先进行代码的解析，生成词法环境用于保存相关数据。程序执行时，对词法环境中的成员进行赋值
### 1.词法环境(LexicalEnvironment)  
  - 用于保存程序中用var定义的变量和声明方式创建的函数  
  - 参考代码
  ```
    var a = 10
    function f(){
		alert(a)
		var a = 5
	}
	f()
  ```
### 2.全局词法环境
  - 在浏览器环境下，全局词法环境是window对象
  - 词法环境中出现重名对象时，函数优先于变量，同样是函数时采用覆盖方式
  - 参考代码
  ```
     alert(f)        //函数和变量同名时优先使用函数
	 alert(g)     //函数同名时，以最后定义的为准，覆盖前面的
	 function f(){console.log('ff')}   
	 var f = 100
	 function g(){console.log('g1')}
	 function g(){console.log('g2')}
  ```
### 3.函数词法环境
  - 函数每次调用前，都会产生一个独立的词法环境
  - 函数词法环境保存函数内部var定义的变量和声明方式创建的内部函数，还有函数传入的参数，以及arguments
  - 函数内部没有用var定义的变量，会作为全局词法环境的成员
  - 参考代码
  ```
    //函数词法环境中包含参数a，b，变量b和函数a，以及arguments
    function f1(a,b) {
           alert(a)        //函数和变量同名时优先使用函数
           alert(b)       //变量同名时，以传入的参数为准
           var b = 100
           function a(){console.log('ff')}
		   c = 500	//没有用var定义的变量,函数调用的解析阶段将保存到全局词法环境中，执行此代码时进行赋值	   
    }
    f1(10,20)
	alert(window.c) 
  ```
### 4.练习题
   说出以下代码的结果  
  ```
    //第一题
    alert(a)        //a解析时全局词法环境中存在，执行时未赋值
    alert(b)        //b解析时全局词法环境中存在，执行时未赋值
    alert(c)        //c解析时全局词法环境中不存在，执行时报错
    alert(d)        //d解析时全局词法环境中不存在，执行时报错
    var a = 8
    if(false){
        var b = 10   //不存在块作用域，b保存在全局词法环境中
    }else{
       c = 100
    }
    function f(){
        var d = 9       //函数调用前，函数词法环境中存在，执行调用时赋值
    }
    //第二题
    alert(a)  	//解析时，a在词法环境中，但没有赋值，显示为undefined	  
    alert(b)    //解析时，b没有在词法环境中，导致报错
    alert(f)    //解析时，f在词法环境中，显示函数的定义代码
    alert(g)    //解析时，g作为变量在词法环境中，显示为undefined
    var a = 20
    alert(a)	//执行时，a已经赋值，显示为20  
    b = 100
    alert(b)    //执行时b直接赋值，显示100  
    function f(){}  
    var g = function(){}
    alert(g)	//执行时，g已经赋值，显示为函数g的定义代码
  ```  
## 二、作用域
### 1.作用域是指一个变量或函数可以被访问的区域范围
### 2.JS的作用域是静态作用域（词法作用域），不存在块作用域或动态作用域  
  - 块作用域是大括号包括起来的范围内有效
  - 参考代码
	```
       for(var i = 0 ; i <10 ;i++){}
	   alert(i)		//i在{}外部还可以被访问,因此js没有块作用域的特性
	```
  - 静态作用域是指在定义的时候就确定下来的区域范围，函数的作用域指向上级的词法环境，由此构成作用域链
  - 参考代码
	```
    var y = 100
    function f(){
      var x = 10
      function g(){
        alert(x)
        alert(y)
      }
      g()
    }
    f()
    (1)f.[scope] -> 全局词法环境(window对象),保存变量y和函数f
    (2)f调用前生成函数词法环境f.le,保存变量x和内部函数g，g.[scope]->f.le
    (3)f调用时，将调用函数g，在调用前生成函数词法环境g.le
	  (4)g调用时，在g.le查找变量x和y，由于不存在，通过g.[scope]查找上级词法环境f.le，如果上级词法环境中没有的话，再通过f.[scope]向上查找知道全局词法环境
     g.le->g.[scope]->f.le->f.[scope]->le,简而言之，在一个函数中找不到变量或函数，就到上一级函数中查找，一直到全局作用域为止
	```
### 3.使用Function创建的函数，不具备作用域链特性
  - 参考案例
  ```
          var x = 1000
          function f(){
                var a = 10
                var x = 20
                var g = function(){
                    alert(a)
                }
                g()
                var h = new Function("","alert(x)") //h函数的作用域始终指向全局LE
                h()
          }
        f()
  ```
### 4.作用域的实战应用 
  利用作用域，可以保存公共访问的变量，减少了全局变量的使用和弊端
  - 参考案例  
  ```
  //利用匿名函数的方式，保存变量并不让外部访问，利用window属性进行公开
  (function(){
	var a = 10		
    var b = 20
    function() f(){
		alert(a)
		alert(b)
	}
	window.f = f
  })()
  f()
  ```   	
## 三、闭包
### 1.闭包的定义
  - 闭包是使用了自由变量的函数，所谓自由变量是外部函数定义的变量
  - 闭包是函数及其引用环境的结合体
### 2.闭包的语法
  - 内部函数使用了外部函数中定义的变量,这些变量会被单独保存到闭包对象中
  - 内部函数使用的自由变量可以不是上一级函数定义的，跨级使用也是算作闭包
  - 内部函数如果没有使用外部的自由变量，不会产生闭包
  - 内部函数不使用return也可以实现闭包
  - 闭包的实现原理是作用域链
  - 参考代码  
  ```
      //案例一
	  function f(){
	  	var num = 10
	    function g(){
	      num++  //内部函数使用了外部函数中的变量num
		  alert(num)
	    }
		g()
	  }
	  f()
	  //案例二
	  function f(){
	  	var num = 10
	    return function g(){
	      num++  	
		  alert(num)
	    }
	  }
	  var f1 = f()
	  f1()   //即便外部函数运行结束，闭包的自由变量也会保存
	  f1()  
  ```
### 3.闭包的用途
  - 减少全局变量
  - 实现封装
  - 减少传参数量
  - 参考案例  
  ```
	//减少全局变量
     var num = 1
     function add(){
         alert(num++) //实现累加，需要把num作为全局变量
     }
     add()
     add()
      //改进方式
    function  add(){
		var num = 1;
		return function g(){
			num++
			alert(num)
		}
    }
	var adder = add()
	adder()
	adder()
    //减少传参数量
    function f(base){
		return function g(max){
			var total = 0
			for(var i = 0 ; i < max ; i++){
				total += i
			}
			return total
		}
	}
	var f1 = f(1)
    alert(f1(3))
	//实现封装
  	(function(){
       var m;
       function SetM(num){
		 m = num
       }
	   function GetM(){
          return m
	   }
	   window.f = SetM
       window.g = GetM
    })()
    f(100)
    alert(g)
  ```
### 4.闭包的查看
  通过浏览器的属性查看，Sources选项卡中，查看Closure，内部有自由变量
### 5.闭包的注意点
  - 闭包使用的自由变量是引用，不是复制，会随着自由变量的变化而改变。
  - 导致逻辑错误
  - 参考代码  
  ```
      //案例一
	  function f(){
	  	var num = 10
	    function g(){
	      alert(num)  //num的数值会变化，不是固定的
	    }
		num++
		g()
	  }
	  f()
	  //案例二
	    for(var i = 0 ; i < 3 ; i++){
                    var ele = document.getElementById(i+1)
                     ele.onclick = function(){
                        alert(i+1)  //不同dom元素单击的结果是一样的，i始终是最后的运算结果，会导致逻辑错误
                     }       
           }
     
  		//改进
		 for(var i = 0 ; i < 3 ; i++){
                    var ele = document.getElementById(i+1)
                     ele.onclick = (function(id){
                        return function(){
                               alert(id+1)
                        }
                     } )(i) 
           }
  ```
## 四、对象
### 1.对象类型
  - JS内置对象（Number）
  - 宿主环境（window）
  - 自定义对象
### 2.对象的创建
  - 对象字面量方式
	  - 大括号包括起来的数据成员,类似于JSON格式（key不用引号）
	  - 数据成员包含属性和方法
	  - get/set型属性
	  - 参考案例
	  ```
		//字面量对象
	    var p = {
            name:'tom',         //属性
            _age:18,			//私有属性
            work:function(){console.log('工作中。。。')},     //方法
            get age(){return this._age},        //特殊类型属性
            set age(val){this._age = val}
        }
        p.age=20
        alert(p.age)
		alert(p.name)
		p.work()
	  ```
  - Object
	  - new Object()方式创建对象，再使用成员访问符进行赋值
	  - 通过defineProperty方式创建对象成员
	  - 通过defineProperties方式创建多个对象成员
	  - 参考案例
	  ```
		var p = new Object()
	    p.name = 'tom'
		/*
		参数1表示需要添加成员的对象
		参数2表示成员的名称
		参数3是成员的特性，是一个字面量对象
		*/
        Object.defineProperty(p,"age",{value:20,writable:false})
		p.age = 100	//无效代码，因为该成员不可改写
		alert(p.age)
		/*
		参数1表示需要添加成员的对象
		参数2是所有的成员，是一个字面量对象	
		*/
		Object.defineProperties(p,{
            age:{
                value:100,
                writable:true
            },
            address:{ 
                value:'address1',
                writable:true
			},
            height:{
                get:function(){return this._height},
                set:function(val){this._height=val}
            }
        })
       alert(p.age + '  ' + p.address)
	   p.height=180
	   alert(p.height)
	  ```
  - 构造函数
### 3.对象成员特性修饰符
  - writable :设置是否可写
  - enumerable ：设置是否可以枚举，用于遍历
  - configurable ：设置是否可以修改成员特性或删除成员
  - 用define系列方法设置时，默认都是false
  - 通过Object.getOwnPropertyDescriptor(对象名,成员名)查看成员的具体特性
### 4.对象的构造器
  - 每个对象都具有一个constructor属性，表示其创建的类型
  - 基本数据类型都由其类型函数构造而成
  - 自定义函数对象由Function构造而成
  - 对象由Object函数构造而成
  - Object以及不同类型函数都由Function构造而成
  - Function仍由Function构造而成
  - 参考案例
  ```
    var a = 1
    var a1 = new Number(1)
    var arr = [1,2,3]
    var p = {}
    var p1 = new Object()
    var f = function(){}
    console.log(a.constructor)   //构造器是Number
    console.log(a1.constructor)   //构造器是Number
    console.log(arr.constructor) //构造器是Array
    console.log(p.constructor)	 //构造器是Object
    console.log(p1.constructor)	 //构造器是Object
    console.log(f.constructor)	 //构造器是Function
    console.log(Number.constructor)//构造器是Function
    console.log(Array.constructor) //构造器是Function
    console.log(Object.constructor)//构造器是Function
    console.log(Function.constructor)//构造器是Function
  ``` 
### 5.对象的基本操作
  - 对象的成员访问
	  - 通过成员访问符访问，允许连缀访问
	  - 通过中括号属性访问,当属性名是非字符类型时比较方便
	  - 对象属性不存在时不报错，返回undefined
	  - 参考案例
	  ```
		//字面量对象
	    var p = {
            name:'tom',         //属性
            address:{
				home:'add1',
				office:'add2'
			}
        }
		alert(p.name)
		alert(p['name'])
        alert(p.address.home)//支持连缀访问
        //
	  ```
  - 遍历对象的所有成员
	  - 通过for(key in 对象)遍历，成员是无序排列的
	  - 通过Object.keys(对象)获取所有成员名(数组格式，可以用来排序)
	  - 对象成员必须具有enuerable特性，才可以被遍历到，否则无法显示
	  - 参考案例
	  ```
	     for(var key in p){
        	alert(p[key])
     	 }
         var keys = Object.keys(p)
		 console.log(keys)  //keys是一个数组

	  ```
  - 判断对象是否具有某个成员 
	  - 通过成员名 in 对象方式判断
	  - 通过hasOwnProperty(成员名)判断
	  - 参考案例
	  ```
	    alert('age' in p)
        alert(p.hasOwnProperty('age'))

	  ```
  - 删除对象的成员
    - 通过delete删除对象属性成员和方法成员，但有的属性无法删除，例如toString等系统成员
  - 对象的检测
    - typeof用于检测类型
    - instanceof用于检测是否是某种类型的实例
    - 参考案例
	  ```
	   var num = 10
       console.log(typeof num)
       var p = {}
	   console.log(p instanceof Object)

	  ``` 
### 4.对象的特性
## 五、继承和多态
