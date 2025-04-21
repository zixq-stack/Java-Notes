# Vue2
## 基础篇

### 初识Vue

下载Vue.js，链接：https://cn.vuejs.org/v2/guide/installation.html。开发版和生产版就字面意思。

开发工具：VSCode。
- VSCode中的插件和设置自行百度进行配置。搜索VSCode初始配置即可，然后后续的需要时百度进行配置即可。

- 给浏览器安装vuejs devtool工具，可以“魔法”上网的话直接google商店搜索Vue即可（注意区分Vue2和Vue3），不能的话去阿里云盘链接: https://www.aliyundrive.com/s/JtHVq5SX3po 



```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>初识vue.html</title>

    <!-- 0、导入vue.js -->
    <script src="../js/vue.js"></script>
</head>
<body>

    <!-- 1、定义一个容器 -->
    <div id="app">
        <h3>第一个vue程序，并使用插值表达式取得值为：{{name}} </h3>
    </div>
</body>

<script>

    // 去除浏览器控制台中的错误信息  可以尝试去掉这个，然后进入浏览器控制台，观察上面报的东西
    Vue.config.productionTip = false;

    // 2、初始化Vue容器
    new Vue({
        el: "#app",
        data: {
            name: "紫邪情",
            age: "18"
        }
    })
</script>

</html>
```

 <img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220109230432977-1121379970.png" alt="image" style="zoom:50%;" /> 



> 小结

1. Vue容器中的代码一样符合html规范，只不过是混入了一些Vue的特殊用法而已，这个容器中的代码被称为：**Vue模板**。
2. Vue实例（new Vue处）和 Vue容器是一 一对应的，即：一个Vue实例只能对应一个Vue容器。
3. 当然：真实开发中只有一个Vue实例，后续会见到。
4. `{{xxx}}` 中的xxx要写js表达式，` {{}} ` 即为插值表达式，且xxx可以自动读取到**data中的所有属性**，一旦data中的数据发生改变，那么页面中使用该数据的地方一样发生改变。

js表达式 和 js代码的区分：

- js表达式：一个表达式会产生一个值，可以放在任何一个需要值的地方。

```javascript
a

a + b

test(2)

x === y ? 'a' : 'b'
```

- js代码：

```javascript
if（）{}

for（）{}

.........
```







### 认识v-bind：数据单向绑定

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>vue之v-bind</title>

    <script src="../js/vue.js"></script>
</head>
<body>

    <div id="app">
        <!-- 常规写法：v-bind 绑定的是属性 -->
        <a v-bind:href="url"></a> 
            <!-- 
				多层嵌套取值：插值表达式   取的东西就是下面Vue实例new Vue({}) data中的东西，可以理解为在找寻集合中的元素
            -->
        <h3>{{person.name}}</h3>
        <!-- 简写 -->
        <h3 :mySex="person.sex">{{person.name}}</h3>        
    </div>
    
</body>

<script>

    Vue.config.productionTip = false;

    new Vue({
        el: "#app",     // el指定的是为哪一个容器服务  值就是一个css中的选择器
        data: {         // data存储的就是数据，这些数据是供el指定的容器去用的
            url: "https://www.cnblogs.com/zixq/",

            person: {
                name: "紫邪情",
                sex: "女"
            }
        }
    })
</script>
</html> 
```





> 小结：Vue模板有两大类

1. 插值语法，前面已经见到了，就是：`{{取data中保存的数据}}`。

- 功能：解析标签体的内容。
- 注意：`{{}}`中的东西必须是js表达式。

2. 指令语法：

- 功能：解析标签（ 包括属性、标签体内容、绑定事件...... )。
- 例子： `v-bind:herf = "xxxx"` 或 简写为  `:herf = "xxxx"`。注意：xxx同样要写成js表达式。
- 另外：Vue中有很多指令语法，形式都是： v-xxxx。如：v-bind、v-model、v-if、v-for、v-on......，而且一些指令也都可以简写。





### 认识Vue的数据绑定

> 这里需要了解mvvm模式，要理解，可以先参照Java中的MVC模式，然后再了解MVVM，具体内容自行先百度了解一下。@紫邪情



```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>v-model</title>

    <script src="../js/vue.js"></script>
</head>
<body>

    <div id="app">
        <!-- 单向绑定：绑的是属性 -->
        单向绑定：<input type="text" :value="name">
        
        <!-- 双向绑定：绑的是值 -->
        双向绑定：<input type="text" :myName="name" v-model:value="username">   
        <!-- 双向绑定的简写 因为：model绑的就是值，因此：value可以省掉 -->
        双向绑定：<input type="text" :myName="name" v-model="username">
    </div>
</body>

<script>

    Vue.config.productionTip = false;

    new Vue({
        el: "#app",
        data: {
            name: "紫邪情",
            username: "邪公子"

        }
    })
</script>
</html>
```



测试单向绑定
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220109234642452-76190818.png" alt="image"  />





测试双向绑定
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220109234818796-2025201519.png" alt="image"  />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220109234853946-1110651492.png" alt="image" />



> 数据绑定小结：Vue中数据绑定有两种方式

1. 单向绑定（v-bind），数据只能从data流向页面。
2. 双向绑定（v-model），数据不仅能从data流向页面，还可以从页面流向data，但需注意如下两点

- 双向绑定一般都应用在表单类元素上，如：input、select等。
- `v-model:value`  可以简写为 v-model，因为：v-model默认收集的就是值。





### el和data的两种写法

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>vue的el和data的两种写法</title>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app"></div>

    <script>
      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        // el:'#app',   // el 常规写法
        // data:{},     // data 常规写法	这种也叫对象式写法

        /* 
        	data 函数式写法，这种写法后续经常见
        	
        	这里使用 data：function(){} 也可以，但是：绝对不能写成 data:()=>{}
        	因为这种写法变质了，成为Window对象了，这样后续用this指向时，这个this所指代的就不是Vue实例了
        */
        data () {
          return {
          }
        }
       });

       // el 另外的写法		这种写法需要记住，后续组件化开发需要用到
       vm.this.$mount('#app')
    </script>
  </body>
</html>
```



> 小结：el和data的两种写法

1. el

- new Vue时配置el选项。
- 先创建Vue实例，然后再通过 `vm.this.$mount( '#app' )` 来指定el的值。

2. data

- 对象式。
- 函数式。
- 如何选择哪种用法：目前都可以，但是后续会玩组件，则：必须用函数式，而且不可以用兰姆达表达式，即：上面的 `data:()=>{}`，否则：会出错。

3. 一个原则：由Vue管理的函数，一定不要写箭头函数，就上面的兰姆达表达式，否则就会导致this不再是Vue实例了。

- 这个this有大用处，在"new Vue"中用了this就是指的当前的Vue实例，后续可以通过这个this玩很多东西 。



### 理解MVVM模型

> M    指：model。
>
> V     指：view。
>
> VM   指：ViewModel。



```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>理解MVVM</title>

    <script src="../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2>姓名:{{name}}</h2> <br>
        <h2>性别:{{sex}}</h2>   
        <hr>
        <h2>智商:{{1+1}}</h2>
    </div>

    <script>
      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            name: "紫邪情",
            sex: "女"
        },
       });

       console.log(vm);
    </script>
  </body>
</html>
```



![image-20240214195725648](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214195655888-524822885.png)



![image-20240214195932420](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214195901931-750088266.png)

 



> 查看代码中的MVVM

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220110005808477-1490200236.png" alt="image" />



> 小结：MVVM模型

1. M：Model（模型） ————> data中的数据。
2. V：View（视图 ）—————> 模板代码。
3. VM：ViewModel（视图模型）————> Vue实例。

观察效果发现

1. data中所有的属性最后都在vm身上，即：ViewModel。
2. vm身上所有的属性 及 Vue原型上的所有属性，在Vue模板中都可以直接使用。原型就是下图中的这个。



 ![image-20240214200151755](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214200121156-16549577.png) 





### 回顾 Object.defineProperty() 函数

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>了解object.defineProperty()</title>
    <script src="../js/vue.js"></script>
  </head>

  <body>
    <script>

      Vue.config.productionTip=false;
      
      let sex = '女'
      let person = {
        name: '紫邪情'
      }

      /* 
      	参数说明：
      			person 为要修改的对象
      			sex	为具体修改的对象中的哪个属性
      			{} 对修改属性的配置
      */
       Object.defineProperty(person , 'sex' , {
        //  以下就是{}中的相关配置
        //  value: '男',
        //  enumerable: true,    // 这个sex是否可以被遍历
        //  writable: true,      // 这个sex是否可以被修改
        //  configurable: true,   // 这个sex是否可以被删除

         // 当有人获取sex这个属性时会触发这个get()方法
         get(){
            console.log('有人读取sex属性了');
            return sex;
          },

          // 当有人修改sex属性的值时会触发这个set方法
          set(value){
            console.log('有人修改了sex的值');
            return sex = value;
          }
       });

    </script>
  </body>
</html>
```



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220114232945298-702502795.png" alt="image" /> 

> 注意：去找temp获取值是调用了`get()`方法进行获取的，相应地`set()`方法也知道是怎么回事了。





### 简单理解数据代理原理

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>简单理解数据代理</title>
  </head>

<body>

	<script>

        let obj1 = {x:100};
        let obj2 = {y:200};

        // 数据代理： 通过一个对象代理 对 另一个对象中属性的操作（ 读 / 写 ）
        Object.defineProperty(obj2 , 'x' , {
          get(){
            console.log("有人获取了x的值");
            return obj1.x;
          },

          set(value){
            console.log("有人修改了x的值");
            obj1.x = value;
          }
        })
    </script>
  </body>
</html>
```



> 原理图：下图是简单了解，深入了解在后面玩Vue监视数据原理时会再次见到`defineProperty()`和这个数据代理。

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115001310441-270063892.png" alt="image" />



> 小结：Vue中的数据代理

- 通过vm对象来代理data对象中的属性的操作（ 读 / 写 ）
- 好处：更加方便操作data中的数据
- 原理：
  - 通过`object.defineProperty()`把data对象中所有属性添加到vm上
  - 为每一个添加到vm上的属性，都指定一个 getter / setter
  - 在getter / setter内部去操作（ 读 / 写 ）data中的属性



### 事件绑定及其修饰符

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>事件绑定及其修饰符</title>

    <script src="../js/vue.js"></script>
  </head>

  <!-- 
        常用的事件修饰: 事件、事件，放在对应事件后就可以了，如：@click.prevent = "xxxx"
            prevent     表示：阻止默认事件的触发
            stop        表示：阻止事件冒泡
            once        表示：只执行一次事件
   -->
  
  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">

        <!-- vue中事件绑定的简单写法 -->
        <button v-on:click="showInfo()">点我显示提示信息</button>

        
        <!-- vue中事件绑定的简写 还可以传递参数 -->
        <button @click="showInfo2($event, 66)">点我获取带参的事件信息</button>
    </div>

    <script>

        Vue.config.productionTip=false;
        
      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{},

        methods: {
            showInfo(){
                alert("这是vue中的事件绑定");
            },

            showInfo2(event , number){
                console.log(event + "=====>" + number);
            }
        }
       });
    </script>
  </body>
</html>
```



> 小结：事件的使用

1. 使用`v-on:xxxx`，或 `@xxx` 来绑定事件，xxxx就是事件名。
2. 事件的回调需要配置在methods对象中，最终会在vm上。
3. methods中配置的函数，记得不要用兰姆达函数，否则this无效了。
4. methods中配置的函数，都是被Vue所管理的函数，this指向的就是vm 或 组件实例对象。
5. `@click = “demo”` 和 `@click = “demo($event)` 效果一致，但是：后者可以传参（后续有用）。



### 计算属性

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>计算属性conputed</title>

    <script src="../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        姓名: <input type="text" v-model="name"> 
        性别: <input type="text" v-model="sex">
        <hr>
        信息: <span>{{info}}</span>
    </div>

    <script>

    Vue.config.productionTip=false;
      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            name:'紫邪情',
            sex:'女'
        },

        // 所谓的计算属性：就是通过已有属性（ 一般为data中的 ）计算得来
        computed: {
            info:{
                get(){
                    console.log('开始调用get()');
                    return this.name + '-' + this.sex
                },

                set(value){
                    console.log('开始调用set()' + value);
                    
                    // 计算属性要被修改，那必须写set()去响应修改，且set()中要引起计算时依赖的数据发生改变
                    const arr = value.split('-');
                    this.name = arr[0];
                    this.set = arr[1];
                }
            }
        }
       });
    </script>
  </body>
</html>
```



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220110164430015-707254161.png" alt="image" />





> 计算属性小结

1. 定义：要用的属性不存在，要通过已有属性计算得来。
2. 原理：借助了object.defineproperty()的getter和setter。
3. `get()`什么时候执行？

- 初次读取时会执行一次。
- 当依赖的数据发生改变时会再次被调用。

4. 优势：与methods相比，内部有缓存机制（复用），效率更高、调试方便。
5. 注意：

- 1、计算属性最终都会在vm（Vue实例）上，直接读取即可。
- 2、若计算属性要被修改，那必须写set()去响应修改，且set()中要引起计算时依赖的数据发生改变（ 例子中没有做这一步 ）。

6. 另外，计算属性是可以简写的，就是把`set()`去掉，因为：更多时候是读取，并不修改，同时修改还要去控制台（刷新就又没了），因此：把`set()`去掉就是计算属性的简写。





### 结合computed、methods、v-on做一个小Demo

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>使用计算属性做一个Demo</title>

    <script src="../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2>你是:{{result}}</h2>

        <button @click="changeResult">切换名字</button>
    </div>

    <script>
       // 去除浏览器控制台中的错误信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            name: true
        },

        computed:{
            result(){
                return this.name?'紫邪情':'紫女'
            }
        },

        methods: {
            changeResult(){
                return this.name = !this.name
            }
        },
       });
    </script>
  </body>
</html>
```

  

 <img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220110201845885-743416982.png" alt="image" style="zoom:50%;" /> 



但是：上面的代码其实是可以简化的

![image-20240214201850977](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214201820783-27091213.png)

我们主要做的是图中的这一步，因此：简化 ———— 衍生出v-on的另外一个玩法

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>使用计算属性做一个Demo 简化</title>

    <script src="../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2>你是:{{result}}</h2>

        <!-- 
			@xxx = 'yyyy' 如果只是做一件简单的事情，那么就可以使用下面的方式
              因为：v-on是可以支持js表达式的	注意：不是js代码
              但是：如果这个v-on需要做多件事，那么最好就别这么玩
                    如：切换了名字，还要弄一个弹窗这种就别玩
        -->
        <button @click="name = !name">切换名字</button>
    </div>

    <script>
       // 去除浏览器控制台中的错误信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            name: true
        },

        computed:{
            result(){
                return this.name?'紫邪情':'紫女'
            }
        },

        // methods: {
        //     changeResult(){
        //         return this.name = !this.name
        //     }
        // },
       });
    </script>
  </body>
</html>
```





### 监视属性

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>监视属性 watch</title>

    <script src="../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2>你是:{{result}}</h2>

        <button @click= "name = !name">切换名字</button>
    </div>

    <script>
       // 去除浏览器控制台中的错误信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            name: true
        },

        computed: {
            result() {
                return this.name?'紫邪情':'紫女'
            }
        },

        /*
        	监视属性 watch  监视的是属性，也就是data中 和 computed中的都可以监视
			现在这种我是用的简写形式 ———— 前提是：只需要使用 handler() 中的东西时，handler后续会用完整形式
		*/
        watch: {
            name( newValue, oldVaule ){ // 表示的是：监视哪个属性
                console.log('属性发生变化了' , newValue , oldVaule);
            }
        },
       });

    //    当然：监视属性还有一种写法，就是利用Vue的内置函数
/*     vm.$watch( 'name', {  // 这里的name就是指监听哪个属性  而且必须是' '引起来的
        handler( newValue, oldVaule ){
            console.log('属性发生变化了' , newValue , oldVaule);
        }
     }) */
    </script>
  </body>
</html>
```

 ![image-20240214202543914](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214202513271-797098096.png) 





> 小结：监视属性 watch

1. 当被监视的属性变化时，回调函数自动调用，进行相关操作。

回调函数：指的是：`handler() `，第一种写法也是可以用handler，只是把watch换成了`$watch`而已，后面步骤其实是一样的，只是我把第一种给简写了。

2. 监视的属性必须存在，才能进行监视。
3. 监视有两种写法

- （1）、new Vue时传入watch配置。
- （2）、通过`vm.$watch`内置函数进行监视。



#### 深度监视 - 需掌握

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>深度监视</title>

    <script src="../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2>你是:{{result}}</h2>

        <!-- 现在这个name就是person里面的了，这就是需要监视多层属性变化 -->
        <button @click= "person.name = !person.name">切换名字</button>
    </div>

    <script>
       // 去除浏览器控制台中的错误信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            person:{
                name: true
            }
        },

        computed: {
            result(){
                return this.person.name?'紫邪情':'紫女'
            }
        },

        // 下面这种是监视属性的完整写法，前面玩的一个是简写形式
        watch: {
            person:{
                // 监视多层结构中所有属性的变化
                deep: true,     // 深度监视 就做了这一步操作而已
            /* 
                Vue中watch默认是不可以监视属性更里层的，
                如：上面person只可以监视person本身这个属性，理解的话，
                可以采用对象空间值来比对，假如：person空间地址是是01x23，
                那么Vue只会监视这个空间变量有没有发生改变，而内层就不可以监视，
                因为：内层中的属性改变了，但是person这个对象本身并没有改变
             */
                 handler(){
                    console.log('属性改变了');
                } 
            }
        }
       });
    </script>
  </body>
</html>
```





![image-20240214202525658](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214202455365-800087856.png)





> 小结：深度监视

1. Vue中的watch默认不监视对象内部值的改变（监视的是一层）。

2. 配置`deep:true`可以监视对象内部值的改变（监视多层）。

注意：

1. Vue自身可以监视对象内部值的改变，但Vue提供的watch默认不可以。

这个可以验证的，使用多层属性，最后是绑定在vm上的，那么在控制台使用vm去改变值，再去观察页面就可以发现效果了，这里是为了解释Vue不是不可以监视多层的改变，只是：watch默认不支持。

2. 使用watch时根据数据的具体结构（属性是一层还是多层），决定是否采用深度监视。



### computed 和 watch的坑

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>computed和watch的坑</title>

    <script src="../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        姓: <input type="text" v-model="firstname"> 
        名: <input type="text" v-model="lastname">

        <hr/>

        信息: {{fullname}}
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            firstname: '紫',
            lastname: '邪情',
            fullname: '紫邪情'
        },

        watch: {
            firstname(val){
                this.fullname = val + this.lastname;
            },

            lastname(val){
                this.fullname = this.firstname + val;
            }
        }
       });
    </script>
  </body>
</html>
```



![image-20240214203336706](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214203306316-989829269.png)



![image-20240214203324774](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214203254714-615396636.png)



用computed实现

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>computed实现</title>

        <script src="../../js/vue.js"></script>
    </head>

    <body>
        <!-- 被 vm 实例所控制的区域 -->
        <div id="app">
            姓: <input type="text" v-model = "firstname">   
            名: <input type="text" v-model = "lastname">

            <hr/>

            信息: {{fullname}}
        </div>

        <script>
            // 去除浏览器控制台中的警告提示信息
            Vue.config.productionTip = false;

            // 创建 vm 实例对象
            const vm = new Vue({
                // 指定控制的区域
                el:'#app',
                data:{
                    firstname: '紫',
                    lastname: '邪情'
                },

                computed: {
                    fullname(){
                        return this.firstname + this.lastname;
                    }
                }
            });
        </script>
    </body>
</html>
```



最后运行的效果都是一样的，而官网中对这二者有着这样的实例演示
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220111172243425-883506409.png" alt="image" style="zoom:50%;" />

但是：虽然看起来没区别，可是computed和watch还是有区别的。

假如：现在我需要让姓改了之后，隔1s之后再显示到信息那里。那么：computed无法做到，但是watch就可以。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>用watch实现异步操作</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        姓: <input type="text" v-model = "firstname"> 
        名: <input type="text" v-model = "lastname">

        <hr/>

        信息: {{fullname}}
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            firstname: '紫',
            lastname: '邪情',
            fullname: '紫邪情'
        },

        watch: {
            firstname(val){
                // 一定需要注意：这里必须使用兰姆达表达式() =>{}
                /* 
                    前面说过，Vue所管理的函数，切记别用兰姆达表达式，只能用函数式
                            这是为了能够通过this拿到vm实例的东西而已
                            
                    但是注意：这里这个setTimeout()定时函数是Vue所管理的吗？
                            不是，而后面需要的val是哪里的？是Vue实例中的，修改的值，是在Vue实例身上
                            所以：想要拿到Vue身上的val怎么弄？页面的展示都是js引擎帮忙去进行操作 / 找寻的
                            因此：利用js引擎做文章，让它找的时候自动去层级查找，它执行到里面的this.fullname = val + this.lastname时
                            会去找this是谁,()=>{}这里是用的兰姆达表达式，这就会指向Window对象，而Window上没有fullname，所以就会
                            往外找，找到firstname(val)这是函数式，指向的就是Vue实例，也就找到了fullname、val.....
                */
                setTimeout( () => {
                    this.fullname = val + this.lastname;
                }, 1000);
            },

            lastname(val){
                this.fullname = this.firstname + val;
            }
        }
       });
    </script>
  </body>
</html>
```

![image-20240214203307177](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214203237002-1037715827.png)

而利用computed就不可以做到上面的这种异步操作

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220111174834292-2087858254.png" alt="image" />





> 小结：computed和watch的区别

1. computed能完成的功能，watch绝对都可以做到。
2. watch能完成的功能，computed不一定可以做到，如：上面举的例子进行异步操作。

原则：

1. 凡是被Vue所管理的函数，最好都写成函数式（ 即：普通函数），这是为了能够让this指向Vue实例 或 组件实例对象（后续会见到），这样就可以通过this拿到它们对应的东西。
2. 凡是不被Vue所管理的函数（如：定时器里面的回调函数、ajax中的回调函数.... ），最好都写成兰姆达表达式（即：箭头函数），这样做也是为了能够让this指向Vue实例 或 组件实例对象。





### 样式绑定

#### class样式绑定

假如有如下的一个页面

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>class样式绑定</title>

    <script src="../../js/vue.js"></script>

    <style>
        body {
          background-image: url(img/bg3.jpg);
          text-align: center;
          background-size: 100% 100%;
          height: 100%;
          overflow: hidden;
          background-repeat: no-repeat;
          background-position: center;
          background-attachment: fixed;
        }

        .top {
          background: #ffffff2e;
          width: 100%;
          position: absolute;
          bottom: 0;
          line-height: 60px;
          left: 0px;
          right: 0px;
          color: #fff;
          text-align: center;
          font-size: 16px;
          font-weight: 600;
        }

        .basicLogin {
          position: absolute;
          top: 16%;
          left: 28.5%;
          width: 40%;
          padding: 70px 2%;
          text-align: center;
        }

        .title {
          font-weight: 600;
          font-size: 22px;
          color: #0000FF;
          margin-bottom: 40px;
        }

        .line {
            border-bottom: 1px solid #ffff;
            margin: 22px 1%;
            width: 96%;
        }

        .line input {
            border: none;
            padding: 0px 1%;
            margin: 1%;
            background: #ffffff14;
            width: 84%;
            font-size: 16px;
            line-height: 30px;
            outline: none;
        }

        .line .smallImg {
          width: 26px;
          float: left;
          vertical-align: middle;
          margin-top: 1px;
        }

        .logBut {
          background: #7bb5ee;
          padding: 10px 80px;
          border: none;
          color: #fff;
          margin-top: 40px;
          font-size: 16px;
          cursor:pointer;
        }
    </style>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">

      <div class="top">&copy;紫邪情&nbsp;·&nbsp;用Java改变未来</div>

      <div class="basicLogin">
        <div class="title">&copy;紫邪情&nbsp;·&nbsp;登录</div>
        <div class="line">
          <img class="smallImg" src="img/icon-4.png" />
          <input placeholder="请输入账号" type="text" />
        </div>
        <div class="line">
          <img class="smallImg" src="img/icon-5.png" />
          <input placeholder="请输入密码" type="password" />
        </div>
        <button type="button" class="logBut">登&nbsp;&nbsp;录</button>
      </div>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            
        },
       });
    </script>
  </body>
</html>
```

![image-20240214203606930](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214203537827-1334812105.png)



> 现在需求1：中间的登录页样式，不要这种，想要换一种：假如是如下这种

```css
.login1 {
    background: #ffffffd6;
    border-radius: 2px;
}
```

![image-20240214203747927](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214203717716-417453652.png)



![image-20240214203816108](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214203746512-1789949797.png)



可是，领导那个灾舅子滴看了效果觉得还是不行，觉得前面一种还行（不确定的样子），此时：增加样式

```css
.login2 {
    transform: translate(-50%,-50%);
    background: rgba(0,0,0,.8);
    box-sizing : border-box;
    box-shadow: 0 15px 25px rgba(0,0,0,.5);
    border-radius: 10px;
}
```



![image-20240214204021975](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214203951581-281233584.png)



![image-20240214204039525](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214204009609-466190511.png)



后面又说其实都可以，但是先把两种都留着，因此：此时这个class的名字不确定到底是哪一个，需要动态去绑定，所以通过Vue来动态绑定一下， 即：把class类名交给Vue托管。
![image-20240214204139762](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214204109376-2131056472.png)



class类名不确定，交由Vue托管很简单，而且知识点前面已经学过了，就是v-bind

![image-20240214204329438](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214204259339-1066054472.png)



```html
  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">

      <div class="top">&copy;紫邪情&nbsp;·&nbsp;用Java改变未来</div>

      <div class="basicLogin" :class = "unknown" @click = "changeLogin">
        <div class="title">&copy;紫邪情&nbsp;·&nbsp;登录</div>
        <div class="line">
          <img class="smallImg" src="img/icon-4.png" />
          <input placeholder="请输入账号" type="text" />
        </div>
        <div class="line">
          <img class="smallImg" src="img/icon-5.png" />
          <input placeholder="请输入密码" type="password" />
        </div>
        <button type="button" class="logBut">登&nbsp;&nbsp;录</button>
      </div>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
          unknown: 'login1'
        },

        methods: {
          changeLogin(){
            if( this.unknown === 'login1'){
              this.unknown = 'login2'
            }else{
              this.unknown = 'login1'
            }
          }
        }
       });
    </script>
  </body>
```

![image-20240214204351572](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214204321929-1230616638.png)



![image-20240214204443756](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214204413730-1557199331.png)





> 需求2：现在class类名的个数、名字都不确定

![image-20240214204541939](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214204511463-565961690.png)



那么就继续改造

![image-20240214204621769](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214204551323-1970950215.png)



```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>数组写法</title>

    <script src="../../../js/vue.js"></script>

    <style>

        body {
          background-image: url(../img/bg3.jpg);
          text-align: center;
          background-size: 100% 100%;
          height: 100%;
          overflow: hidden;
          background-repeat: no-repeat;
          background-position: center;
          background-attachment: fixed;
        }

        .top {
          background: #ffffff2e;
          width: 100%;
          position: absolute;
          bottom: 0;
          line-height: 60px;
          left: 0px;
          right: 0px;
          color: #fff;
          text-align: center;
          font-size: 16px;
          font-weight: 600;
        }

        .basicLogin {
          position: absolute;
          top: 16%;
          left: 28.5%;
          width: 40%;
          padding: 70px 2%;
          text-align: center;
        }

        .login1 {
          background: #ffffffd6;
          border-radius: 2px;
        }

        .login2 {
          background: rgba(0,0,0,.8);
          box-sizing : border-box;
          box-shadow: 0 15px 25px rgba(0,0,0,.5);
          border-radius: 10px;
        }

        .title {
          font-weight: 600;
          font-size: 22px;
          color: #0000FF;
          margin-bottom: 40px;
        }

        .line {
            border-bottom: 1px solid #ffff;
            margin: 22px 1%;
            width: 96%;
        }

        .line input {
            border: none;
            padding: 0px 1%;
            margin: 1%;
            background: #ffffff14;
            width: 84%;
            font-size: 16px;
            line-height: 30px;
            outline: none;
        }

        .line .smallImg {
          width: 26px;
          float: left;
          vertical-align: middle;
          margin-top: 1px;
        }

        .logBut {
          background: #7bb5ee;
          padding: 10px 80px;
          border: none;
          color: #fff;
          margin-top: 40px;
          font-size: 16px;
          cursor:pointer;
        }

    </style>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">

      <div class="top">&copy;紫邪情&nbsp;·&nbsp;用Java改变未来</div>

      <div class="basicLogin" :class = "unknown" @click = "changeLogin">
        <div class="title">&copy;紫邪情&nbsp;·&nbsp;登录</div>
        <div class="line">
          <img class="smallImg" src="../img/icon-4.png" />
          <input placeholder="请输入账号" type="text" />
        </div>
        <div class="line">
          <img class="smallImg" src="../img/icon-5.png" />
          <input placeholder="请输入密码" type="password" />
        </div>
        <button type="button" class="logBut">登&nbsp;&nbsp;录</button>
      </div>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
          unknown: [ 'login1' , 'login2' ]
        },

        methods: {
          changeLogin(){
            
            const arr = [ 'login1' , 'login2' ];
            this.unknown = arr[ Math.floor( Math.random() *2 ) ];
          }
        }
       });
    </script>
  </body>
</html>
```



![image-20240214204658008](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214204627864-769498681.png)



> 需求3：要绑定的class名字确定，但是：个数不确定，也需要动态绑定

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220112163418216-1474875259.png" alt="image" />



![image-20240214204808081](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214204737922-1384653019.png)







> 小结：class样式绑定

1. 字符串写法，适用于：class名字不确定时使用。
2. 数组写法，适用于：class名字、个数都不确定时使用。
3. 对象写法，适用于：class名字确定、但个数不确定，如：例子中的login1可能用，可能不用，而login2也是一样，所以这就是各种组合。





#### style行内样式绑定 - 了解

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220112181039986-2086408185.png" alt="image" />



![image-20240214204951831](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214204921549-158727270.png)





> 小结：class和style样式绑定

1. class样式绑定

- 写法    `:class = "xxxx"`，其中xxxx可以使字符串、数组、对象。
- 字符串写法适用于：类名不确定、要动态获取（ 交由Vue管理，随时调试 ）。
- 数组写法：要绑定多个样式、class名字和个数都不确定。
- 对象写法：要绑定多个样式、class名字确定而个数不确定（ 个数不确定是因为不知道某个样式用不用 ）。

2. style样式绑定

- 写法  `:style = “xxx"`，其中xxx可以是对象、数组。
- 对象写法是推荐用的一种。

3. 原则：在Vue中静态不变的东西，就放在Vue模板中即可（ 即：那个div容器中 ），而动态改变的东西就放在Vue实例身上。





### 条件渲染 v-if

#### v-if

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>v-if</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <!-- 
			v-if和java中的if一样，只要结果为true就执行，为false就不执行
            所以：这里面不用true，用 1 === 1 也行，当然：使用v-bind绑定，写到data中，只要最后结果为boolean值即可
            相应地：有了v-if，那当然也有v-else-if、v-else
				不过 v-else 这个有点特殊
            		这是什么条件都不满足的时候，最终执行的		类似于try....catch....finally中的finally
        -->
        <div v-if = "true">
            <img :src="result" alt="不见了影响你开法拉利不？" :style="obj">
        </div>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            result: 'img/19.jpg',
            obj: {
                width: '500px',
                hight: '500px'
            }
        },
       });
    </script>
  </body>
</html>
```



![image-20240214205400676](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214205330764-462473229.png)



![image-20240214205426989](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214205356575-1036055683.png)



![image-20240214205507638](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214205437737-1863526041.png)



> 注意：v-if、v-else-if、v-else组合用时，需要留意代码是紧挨着的即可。这就好比：Java的pageHelper分页插件，用startPage和pageInfo联合使用一样，数据查询完了之后放到pageInfo<>()中，这二者中间就不可以夹杂另外的语句，否则报错。









#### v-if配套工具template

> 注意事项：template“只能”与v-if搭配使用。



在弄这个之前，回到前面的v-if代码

![image-20240214205718574](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214205648241-509941132.png)



![image-20240214205752098](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214205722110-1067556229.png)



所以：使用v-if和template改造

![image-20240214205811126](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214205740672-826578790.png)



![image-20240214205832588](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214205802627-599573187.png)







### v-show

v-if会了，那么v-show就会了，用法和单纯的v-if一模一样。

不过它和v-if的原理不一样。

![image-20240214205944158](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214205913746-767389780.png)



![image-20240214205958662](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214205928137-901411021.png)







> 小结：v-if和v-show

1. v-if

写法：

```html
v-if = “表达式”

v-else-if = “表达式”

v-else = “表达式”
```

适用于：切换频率较低的场景，因为：v-if是直接把不展示的DOM元素给移除掉。

![image-20240214210151451](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214210120859-1230557210.png)

特点：不展示的DOM直接移除。

注意：v-if可以和v-else-if、v-else一起使用，但是：要求结构不可以被“打断”（代码紧挨着）。

2. v-show

- 写法：`v-show = “表达式”` 。
- 适用于：切换频率较高的场景。
- 特点：不展示的DOM元素未被移除，仅仅是使用`display:none`样式给隐藏掉了而已。

3. 注意：使用v-if时，元素可能无法获取到，而使用v-show，则：一定可以获取到。

因为：v-show是做了样式修改，但是DOM节点还在，所以是可以操作这个DOM节点的，但是：v-if是直接把DOM元素给移除掉了，万一是误操作而导致把v-if的值弄为false了，那后续万一其他地方用到了v-if移除的节点呢？不就裂开了吗。





### 列表渲染 v-for

#### 认识v-for

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>认识v-for</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <!-- 1、遍历数组 -->
        <h2>人员信息</h2>
        
        <!-- 简写形式 -->
        <ul>
			<!-- 
                v-for就和js中的for in差不多
                :key="p.id"  此处：是给每一个li节点绑定一个唯一的id
									和身份证号一样 就相当于是<li id = ‘xxx’>，此时不写没事，但是最好都带上，后续有用
                p  代表的就是：从persons中遍历出来的每一条数据
             -->
            <li v-for = "p in persons" :key="p.id">
                {{p.name}} ----- {{p.age}}
            </li>
        </ul>

        
        <!-- 完整写法 -->
        <ul>
            <li v-for = "(val,index) in persons" :key="val.id">
                {{val.name}} ----- {{val.age}} ------ {{index}}
            </li>
        </ul>


        <!-- 2、遍历对象 -->
        <h2>神奇之地</h2>
        <ul>
            <li v-for = "(val,index) in like" :key="index">
                {{val}} ------ {{index}}
            </li>
        </ul>

        <!-- 另外：v-for还可以遍历字符串、某个数字（循环这个数字这么多次），但是这两种基本上都不用，所以不说明了 -->


    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            persons: [
                // 数组中又放得有对象
                {'id':'10001','name':'紫邪情','age':'18'},
                {'id':'10002','name':'紫女','age':'18'},
                {'id':'10003','name':'邪公子','age':'19'}
            ],

            like: {
                name: '香牌坊',
                ko:'范冰冰'
            }
        },
       });
    </script>
  </body>
</html>
```



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220112222319358-81596417.png" alt="image" s/>





> 小结：v-for

- 语法：`v-for = “ ( intem，index)  in   xxx   ”:key = “yyy”` 。
- 可遍历：数组、对象、字符串（用得少）、指定次数（用得更少）。
- 可用于：展示列表数据。





##### key值的坑（作用与原理 ）

在前面v-for中key值使用index时有一些坑的。

> 正确的做法是：最好使用数据的id作为key值，但不是说index作为key值就是错的，前面演示过了，是正常效果，可是：在特定的场景下使用` :key = "index"` 就会出问题。@紫邪情

![image-20240214210916127](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214210845620-2123631113.png)



实例：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>key值的坑</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2>人员信息</h2>
        <ul>
        	<!-- 这里使用index作为key值 -->
            <li v-for = "(p , index) in persons" :key="index">
                {{p.name}} ----- {{p.age}} <input type="text">
            </li>
        </ul>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            persons:[
                {'id':'001' , 'name':'张三','age':'18'},
                {'id':'002' , 'name':'李四','age':'19'},
                {'id':'003' , 'name':'王五','age':'20'}
            ]
        },
       });
    </script>
  </body>
</html>
```

![image-20240214211014602](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214210944277-629930478.png)





现在需要在做一件事情
![image-20240214211129680](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214211059571-1612590268.png)





为什么会发生上面的问题？那就需要看看原理图，从而了解用index做key值的坑了。
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220113000927297-1907936111.png" alt="image" />



了解了上面的index作为key值，那么用数据id作为key值也懂了
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220112232745682-1985254966.png" alt="image" />

> 不写key的话，Vue做了一件默认的操作，就是把遍历的index作为了key值，也就是和前面使用index作为key值是一样的效果了。







> 把以上的内容，换成文字小结一波：Vue中v-for的key有什么作用（原理是什么？）

1. 虚拟DOM中key的作用

- key是虚拟DOM对象的标识，当状态中的数据发生改变时，Vue会根据【 新数据 】生成【 新的虚拟DOM 】。
- 随后Vue进行【 新虚拟DOM 】 与【 旧虚拟DOM 】的差异比较（对比算法 / diff算法）。

2. 对比算法的规则

- 1）、旧虚拟DOM中找到了与新虚拟DOM相同的key。
  - 若虚拟DOM中内容没变，则：直接使用之前的真实DOM；
  - 若虚拟DOM中内容变了，则：生成新的真实DOM，随后替换掉页面中之前的真实DOM。
- 2）、旧虚拟DOM中未找到与新虚拟DOM相同的key。
  - 直接创建新的真实DOM，虽然渲染到页面。

3. 用index作为key值可能会引发的问题

- 1）、若对数据进行：逆序添加、逆序删除.....等破坏数据顺序的操作，则：会产生不必要的真实DOM更新，即：界面显示没问题，但：效率低（效率低是因为：最后生成真实DOM时，文本节点是重新生成的）。

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220113002736388-97542773.png" alt="image" />

所谓的逆序操作就是数据添加在前面，如：例子中是添加在“张三”前面的，但是：在最后添加（ 即：在例子中“王五”后面添加数据，这种在最后面添加，则：在新虚拟DOM中生成真实DOM时，顺序是对的，不会出现上述问题的 。

![image-20240214211826771](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214211913904-1362336383.png)

![image-20240214212027689](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214211957319-325175813.png)



- 2）、若结构中还包含输入类（如：例子中的input）的DOM，则：会产生错误DOM更新 即：页面中数据错位。

4. 开发中如何选择key？

- 1）、最好使用数据自己的唯一标识作为key值，比如：手机号、身份证号.....（直接利用数据库中的主键id也行）。
- 2）、如果不存在对数据逆序添加、逆序删除等破坏数据顺序的操作，只用来渲染列表页面，则：使用index作为key是莫问题的。



### 列表过滤（查询功能 ）

1. **使用watch实现**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>列表过滤（ 查询功能 ）</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <H2>人员信息列表</H2>

        搜索:<input type="text" placeholder="请输入搜索词" v-model="keyWord">
        <ul>
            <li v-for = " p in filterPersons" :key="p.id">
                {{p.name}} -------- {{p.age}}
            </li>
        </ul>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            // 1、搜集搜索框中的内容 搜索框内容变，这里的内容也要变，所以：双向绑定v-model
            keyWord: '',

            // 2、准备初始数据
            persons: [
                {'id':'10001','name':'紫邪情','age':'18'},
                {'id':'10002','name':'紫邪晴','age':'19'},
                {'id':'10003','name':'邪公子','age':'20'},
                {'id':'10004','name':'公孙策','age':'21'},
                {'id':'10005','name':'孙悟空','age':'22'}
            ],

            // 4、需要准备另一个容器来装过滤之后的数据
            filterPersons: []
        },

        // 3、使用watch来实现搜索效果
        /* 
            需要做两件事
                （1）、拿到data的keyWord中的内容
                （2）、使用keyWod中的内容 去 persons中进行过滤筛选，从而把结果渲染到页面
        */
       watch: {
           keyWord: {
               // 这个的作用：页面初始化时就执行一次handler()
               immediate: true,

               handler(newVal){
                   // 1）、过滤数据 把结果放到filterPersons中去
                   this.filterPersons = this.persons.filter( (p)=>{
                       // 没在数组中indexOf()返回的就是-1
                        return p.name.indexOf(newVal) !== -1
                    })
               }
           }
       }
       });
    </script>
  </body>
</html>
```

2. **使用computed实现**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>列表过滤（ 查询功能 ）</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <H2>人员信息列表</H2>

        搜索:<input type="text" placeholder="请输入搜索词" v-model="keyWord">
        <ul>
            <li v-for = " p in filterPersons" :key="p.id">
                {{p.name}} -------- {{p.age}}
            </li>
        </ul>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            // 1、搜集搜索框中的内容 搜索框内容变，这里的内容也要变，所以：双向绑定v-model
            keyWord: '',

            // 2、准备初始数据
            persons: [
                {'id':'10001','name':'紫邪情','age':'18'},
                {'id':'10002','name':'紫邪晴','age':'19'},
                {'id':'10003','name':'邪公子','age':'20'},
                {'id':'10004','name':'公孙策','age':'21'},
                {'id':'10005','name':'孙悟空','age':'22'}
            ],
        },

        // 3、使用computed来实现搜索效果
        /* 
            需要做两件事
                （1）、拿到data的keyWord中的内容
                （2）、使用keyWod中的内容 去 persons中进行过滤筛选，从而把结果渲染到页面
        */
       computed: {
           filterPersons() {
               return this.persons.filter( (p)=>{
                   return p.name.indexOf( this.keyWord ) !== -1
               })
           }
       }
       });
    </script>
  </body>
</html>
```







### 列表排序

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>列表排序</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2>人员信息</h2>
        搜索: <input type="text" v-model = "keyWoed" placeholder="请输入关键字">
        <button @click = "sortType = 1">升序</button>
        <button @click = "sortType = 2">降序</button>
        <button @click = "sortType = 0">原顺序</button>
        <ul>
            <li v-for = " p in filterPersons" :key="p.id">
                {{p.name}} ----- {{p.age}}
            </li>
        </ul>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            keyWoed: '',
            sortType: 0,
            persons: [
                {'id':'10001','name':'紫邪情','age':'18'},
                {'id':'10002','name':'紫邪晴','age':'14'},
                {'id':'10003','name':'邪公子','age':'121'},
                {'id':'10004','name':'公孙策','age':'21'},
                {'id':'10005','name':'孙悟空','age':'42'}
            ]
        },

        computed: {
            filterPersons(){
                /* 
                    排序：就是将列表过滤之后的数据 根据特定字段排序之后，再渲染到页面中即可

                    因此：return this.persons.filter这里不能直接用return，不然就直接把过滤的数据渲染到页面了
                    所以：再对这一步做一下排序操作
                */
               // 先用个数组装起来
                const arr =  this.persons.filter( (p)=>{
                    return p.name.indexOf( this.keyWoed ) !== -1;
                })

                // 然后对数组进行排序操作,但是得先知道用户是点击的哪个按钮（ sortType ）
                if (this.sortType) {
                    arr.sort( (previous,last)=>{
                        return this.sortType == 1 ? previous.age - last.age : last.age - previous.age;
                    })
                }

                return arr;
            }
        }
       });
    </script>
  </body>
</html>
```

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220113155825160-1555011014.png" alt="image" />



有个注意的地方：
![image-20240214212519560](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214212449419-313447450.png)







### Vue监视数据的原理

#### 监视数据失效实例

> 先来看看Vue检测属性失效的问题，从而引申出Vue监视数据的原理。

先来玩正常版

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>监视数据失效实例</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2>Vue监视数据失效问题</h2>
        <ul>
            <li v-for = "p in persons" :key="p.id">
                {{p.name}} ---- {{p.age}}
            </li>
        </ul>

        <button @click = "update">修改数据</button>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            persons: [
                {'id':'10001','name':'紫邪情','age':'18'}
            ]
        },

        methods: {
            update(){
                this.persons[0].name = '紫女';
            }
        }
       });
    </script>
  </body>
</html>
```

![image-20240214212906334](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214212836256-956468014.png)



把代码稍微变一下，整出bug

![image-20240214212935903](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214212905673-340058711.png)



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220114223832941-521386425.png" alt="image" />





<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220114224103658-1294000854.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220114224204952-1585684165.png" alt="image" />





上面的问题是怎么回事？要知道这个的原因就需要了解一下Vue监视数据的原理了，可是装数据的类型有什么？对象 `{ }` 和 数组 `[ ]` ，所以：这里又得去了解这两种类型的原理，当然：像什么data中还可以放字符串这就不用说明了，了解了对象和数组，字符串也就可以反应过来了。



#### Vue监视对象的原理

玩这个就需要又回到前面玩过的“数据代理”了
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115001310441-270063892.png" alt="image" />



前面就说过，这个数据代理，只是简单了解而已，上面的流程其实不算对，因为少了一个步骤，整个流程应该是：

1. 自己写的data。
2. 对data进行加工。
3.  把data中的数据给 `_data` 。
4. 再把 `_data` 的内容给页面需要的data中。即：这一步相当于执行了 `vm._data = data` , 这个data就是页面上的data嘛，所以就成功地让页面中的数据跟着自己的修改而改变了，但是底层不是用 `vm._data = data`，方法名不一样，而且还要更复杂一点。



来回顾一下例子：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>回顾数据代理</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2> 姓名: {{person.name}} </h2>  
        <h2> 性别: {{person.sex}} </h2>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            person: {
                name: '紫邪情',
                sex: '女'
            }
        },
       });
    </script>
  </body>
</html>
```



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115010104945-829779763.png" alt="image" />



根据上面的原理，简单模拟一个vue的数据监视。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>模拟Vue数据监视</title>

  </head>

  <body>
    <script>

        // 准备一个对象
        let data = {
            name: '紫邪情',
            sex: '女'
        }

        // vue做了精妙之一就是：设计了一个构造方法Observer(传对象名),这个东西刚刚在_data中见过它，打开就是用Observer打开的
        function Observer(obj) {
            // 拿到data中的所有key值（ 即：上面的name、sex ），放到一个数组里面去
            const keys = Object.keys( obj );

            // 遍历这些key值，得到每一个key
            keys.forEach( (key) => {
                
                // 使用defineProperty()做数据代理  这里的this就是指：传的对象
                Object.defineProperty( this, key , {
                    get(){
                        return obj[key];
                    },

                    set(val){
                        console.log( "值被修改了，要开始解析数据、生成虚拟DOM、进行对比算法、响应数据了");
                        obj[key] = val;
                    }
                })
            });
        }

        // 调用设计的构造方法，从而开始做 data 转 _data
        const objs = new Observer(data);
        let vm = {};
         vm._data = data = objs

        console.log( objs );

    </script>
  </body>
</html>
```

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115013236412-2057941816.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115013650886-1109383157.png" alt="image" />



当然：上面只是简单流程，甚至多层对象的data加工，一个对象中又有另一个对象 / 数组，这样递归下去，所以Vue做得完善一点就是再对对象进行了递归，直到不是对象了为止，同时：这个例子也没有弄“数据修改再次解析模板、生成虚拟DOM、响应数据的事 ，研究得明明白白，全整得出来的话，我还坐起，早飘了！！！。





##### 了解 Vue.set( target ,key , val )

实例：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>了解Vue.set( target, key, val )</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2> 姓名: {{person.name}} </h2> 

        <h2> 性别: {{person.sex}} </h2> 
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
           person:{
                name: '紫邪情',
                sex: '女'
           }
        },
       });
    </script>
  </body>
</html>
```

现在需要添加一个属性 `age: 18`，不可以直接添加在data中，这样就没意义了。

![image-20240214213802671](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214213732764-518264071.png)



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115174427476-1840478253.png" alt="image" />



当然：还有一种方式也可以实现
![image-20240214213848204](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214213817969-1947118564.png)



上面这是正常情况，都说了target这个参数有坑，官网有介绍：
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115171453820-283154829.png" alt="image" style="zoom:50%;" />



就来玩一下这个坑吧

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>target参数的坑</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2> 姓名: {{name}}</h2> 
        <h2> 性别: {{sex}}</h2> 

        <h2 v-if = "age"> 年龄: {{age}}</h2>
        <button @click = "addAge">添加年龄属性</button>

    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            // 把这里的属性换一下位置，不用person{}对象包起来，直接放到data中
            name: '紫邪情',
            sex: '女'
        },

        methods: {
            addAge(){
                this.$set( this.data, 'age', 18);
            }
        }
       });
    </script>
  </body>
</html>
```

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115175344268-1258729216.png" alt="image" />



> ==这就是target这个参数的坑：target不可以直接指向vm这个实例对象，更不可以把要操作的对象指向data这个根数据对象上==。





#### Vue监视数组数据的原理

写个正常例子，去控制台看数组的结构

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>正常实例 - 控制台查看结构</title>

    <script src="../../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2>爱好</h2>

        <ul>
            <li v-for = "(h , index) in hobby" :key="index">
                <h2> {{h}} </h2>
            </li>
        </ul>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            hobby: [ '吃', '喝', '嫖', '赌', '要不得']
        },
       });
    </script>
  </body>
</html>
```

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115215609173-2011270556.png" alt="image" />



那么我们在控制台把数据改了呢？会不会响应到页面上？
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115215808314-875962506.png" alt="image" />



上面这个问题就和前面一开始的问题一样了：监视数据失效的问题
![image-20240214214033146](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214214002883-1354242197.png)



这二者都是使用的"索引"来找的数组的具体某个值，而这：恰恰就是一个坑，想一下在Vue中我们操作数组是怎么做的？是通过数组调了对应的API，如：`arr.push()`、`arr.shift()`........，那Vue怎么知道我调的是哪个API，换句话说：它怎么知道这个API是做什么的 / 这个API是数组中的，要搞这个问题，就要看看在js中是怎么去调数组的API的？
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115224116579-820904246.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115224447037-1934327502.png" alt="image" />



试验一下：这7个API是否可以做到监视数据的效果
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115224703981-585808273.png" alt="image" />



答案肯定不是
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115225756543-1374639087.png" alt="image" />



而上面探索的结果，在Vue官网说的有
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220115230124408-1456490264.png" alt="image" />



>  所以：Vue中要监视到数组的变化，调用下面的7个API中的一个就可以

```javascript
push()

pop()

shift()

unshift()

splice()

sort()

reverse()
```

一定是上面的7个API中的一个才可以吗？

- 答案也肯定不是，前面玩了 `Vue.set( target, key, val )` 和` vm.$set( target, key, val )` 也可以做到啊，只需要把target转成对应的对象即可（只要找他数组中要监视到的对象就行嘛）。





#### Vue监视数据原理总结

1. vue会监视data中所有层级的数据。
2. Vue如何监视对象中的数据？

- 通过setter实现监视，且要在new Vue时就传入要监视的数据。
- 1）、对象中后追加的属性，Vue默认不做响应式处理。
- 2）、如果需要对后添加的属性做响应式，则：通过如下API就可做到

```javascript
vue.set( target, key, val )			说明一下：其中的key可以是属性名，也可以是数组的下标值

vm.$set( target, key, val )			key和上面一样
```

3. Vue如何监视数组中的数据？

- 通过封装 / 包裹数组更新元素的方法来实现，这个封装的方法做了两件事
  - 1）、调用数组原生的同名API，对数组进行更新；
  - 2）、重新解析数据、生成新的虚拟DOM、进行对比算法、响应页面。

4. 在Vue中要修改数组中的某个元素时，只能使用如下的方法

```javascript
1）、push()、pop()、shift()、unshift()、splice()、sort()、reverse()

2）、vue.set( target, key, val )  或 vm.$set( target, key, val )
```

> 注意：`vue.set( target, key, val )`  和 `vm.$set( target, key, val )` 不能给vm（即：vue实例） 或 根数据对象data 进行添加属性操作。

5. 前面分析对象的原理时说过一个流程：数据发生改变，是通过setter来获取改变的数据，从而重新解析模板、生成新的虚拟DOM、进行对比算法、响应页面。在这里面有一个重要的流程：

- 1）、数据改变、setter获取到改变后的值；
- 2）、setter重新解析模板，**这里1 ——> 2这一步有个专业的词叫做：数据劫持**，劫持劫持嘛，半路把东西拿走了呗（就是setter把数据劫走了，它去做后续的操作）。





### input输入框中v-model的使用技巧

1. **text类型**

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116171101656-2126823594.png" alt="image" />


![image](https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116171020487-980699916.png)



2. **radio单选框类型**

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116171527614-1476176947.png" alt="image"/>



**原因：**

- 1、v-model收集的是value值。
- 2、而radio是选择，是使用checked = true / false来做到是否选中的。
- 3、因此：给此种radio类型的input添加一个value属性即可。

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116172204753-196023706.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116172239470-390865114.png" alt="image" />



3. **checkBox多选框类型**

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116173112153-198098716.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116173132147-930490621.png" alt="image" />



4. **select与option下拉框类型**

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116174648954-972558298.png" alt="image"/>



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116174711783-361997951.png" alt="image" />



5. **textarea类型**

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116175052022-1717499598.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116175022307-412871953.png" alt="image" />



6. **checkBox的另一种类型：做协议勾选的**

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116175635703-449697849.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116175657433-1994104312.png" alt="image" />



7. **做了上面这些就可以把数据收集起来了**

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116180142650-620973224.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116180305466-1333261129.png" alt="image" />





8. **v-model的修饰符**

- 前面说过事件的修饰符。

```javascript
prevent     表示：阻止默认事件的触发

stop        表示：阻止事件冒泡

once        表示：只执行一次事件
```

- 而v-model也有修饰符，就3个而已：number、lazy、trim，看名字大概就知道是干嘛的。



**1）、number修饰符，这个修饰符一般都是和input的number类型一起使用的，如：增加一个年龄。**
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116181337068-1216678639.png" alt="image" />



去控制台查看，会发现一个问题。
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116181811862-473562521.png" alt="image" />



因此：需要把number弄为number类型。

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116181928465-111276390.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116181952400-844435706.png" alt="image" />



**2）、lazy修饰符，这个修饰符就光标离开之后再收集数据，拿textarea来举例。**
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116182223369-58247962.png" alt="image" s/>



所以这种情况应该用lazy来修饰，让光标离开当前输入框之后再收集数据。

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116182331848-426421579.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116182430539-824002411.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220116182501027-1101166697.png" alt="image" />



**3）、trim修饰符，这个前后端都知道，就是去除前后的空格，不演示了**。







> 小结：使用v-model收集表单数据

1. 若： `< input type = "text" />` ，则：v-model收集的是value值，而用户输入的就是value值。
2. 若：`< input type = "radio" />` ，则：v-model收集的是value值，且要给标签配置value属性。
3. 若：`< input type = "checkBox" /> `

- 1）、没配置input的value属性，那么收集的就是checked（true / false，是否勾选）。
- 2）、配置input的value属性：
  - （1）、v-model的初始值是非数组，那么收集的就是checked。
  - （2）、v-model的初始值是数组，那么收集的就是value组成的数组。

4. v-model的3个修饰符

```javascript
number			将输入的字符串转为 有效的数字

lazy			失去光标焦点才收集数据

trim			去掉首尾的空格
```







### 过滤器filters

这里有一个过滤器的知识，使用computed、methods也可以实现过滤器的东西，比如：把时间戳弄成规定的格式，所以不了解也罢，需要时再看都可以，就是另类的函数、而且还支持多个过滤器一起用、以及配置局部和全局的过滤器罢了。





### Vue内置指令

回顾一下：已经学了Vue的哪些内置指令。

```javascript
v-on

v-bind

v-model

v-if 、v-else-if、v-else

v-show
```

另外还有哪些内置指令？

```javascript
v-text

v-html

v-cloak

v-once

v-pre

以及可以自定义指令
```





#### v-text指令

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>v-text指令</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
      <!-- 插值表达式 -->
      <h2> {{name}}</h2>

      <!-- v-text指令 
            这和java中的springboot推荐的thymeleaf模板中的th:text很像
      -->
      <h2 v-text="name"></h2>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
          name: '紫邪情'
        },
       });
    </script>
  </body>
</html>
```

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117141146517-1877607688.png" alt="image" />





但是：v-text和插值表达式有区别

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117141500053-1650656024.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117141553610-86360926.png" alt="image" s/>



> ==插值表达式和v-text指令的最大区别，v-text是将v-text的值去找寻之后，把内容回填到对应的标签里面，是内容全覆盖，所以：例子中就算写了“信息*两个字，但是：也会被v-text值的返回内容给全覆盖了==。



当然：还有一个注意点

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117144000511-709968494.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117144112522-1558704948.png" alt="image" />



> ==v-text和插值表达式都不可以解析html标签==。







> 小结：v-text内置指令

- 作用：向其所在的节点中渲染“文本”内容
- 与插值表达式的区别：v-text会替换掉节点中的所有内容，而{{xxx}}插值表达式则不会





#### v-html指令 - 慎用

先说用法，其实和v-text差不多，但是内在区别很大

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117155453536-682551068.png" alt="image" />

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117155710478-2050508591.png" alt="image" />

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117155802671-263346955.png" alt="image" />



**用法简单，但是：谨慎用就体现在可以解析html标签上，不安全**：

- 这里涉及到cookie的机制。
- 去访问一个需要输入信息的网站时（如：要输入用户名、密码之类的重要信息），信息在你要登录的网站的服务器上核对成功之后，服务器返回的除了会把用户名、密码这些后续需要的东西返回回来之外，还会返回一些服务器响应的特殊json字符串，从而凭借服务器返回的这些信息，你才可以进入到相应的页面，而这些信息就保存在浏览器的cookie中（实现机制不一样保存不一样），如下图：



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117160551652-403063818.png" alt="image"/>



- 而服务器返回的json信息有大用处，你的重要信息就保存在cookie中，比如：成功登录之后，操作其他的功能就不会要求再次输入用户名、密码之类的（个别除外，那是另外设计机制），这里就会想到另一个东西：跨浏览器不会出现无账号密码就可以访问，比如：我在google浏览器登录之后，再用Edge浏览器打开，那Edge是不会登录的，但是：利用v-html就可以做到窃取你在google浏览器中登录之后服务器给你返回的json信息，然后把这些json信息放到Edge浏览器中，那么Edge浏览器就不需要做登录操作，一样可以登录进去。
- 而使用v-html能够获取到cookie是因为：可以在用v-html获取的内容中夹杂一些获取你当前服务器的cookie，然后拼接到窃取人的服务器地址的后面（比如：网页中的恶意广告，那广告词中的词条是通过v-html渲染上去的，那么一点击就会把你电脑上的cookie获取到然后转发到他自己的服务器上去）。
- 当然：现在好一点的浏览器都为了防止xss攻击（模拟用户之手），所以：对浏览器做了一些安全限定，最典型的就是google浏览器，默认是不可以携带cookie转发的，但是有些做的不好的浏览器（网站）就可能没弄这些，一点然后跳过去就遭殃了。







> 小结：v-html

- 作用：向指定节点中渲染包含html结构的内容。
- 与插值表达式的区别
  - 1）、v-html会替换掉节点中所有的内容，而插值表达式{{xx}}则不会。
  - 2）、v-html可以识别html结构。
- 特别注意：v-html有安全性问题
  - 1）、在网站上动态渲染任意html是非常危险的，容易导致XSS攻击。
  - 2）、一定要在可信的内容上使用v-html，此指令永远不要用在用户提交的内容上!!!!!!。





#### v-cloak指令

> 这是专门用来处理页面渲染时因网络或其他原因导致页面无法及时刷新出来，把页面加载过程给显露出来了（等到后续懂了Vue的生命周期时，此命令会更容易懂 ，当然：现在也很容易懂）。@紫邪情



实例：把加载js的代码移动一个位置。
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117230727939-1295155564.png" alt="image" />



然后把浏览器的网速调一下。
![image-20240214220829548](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214220800292-1746438824.png)



然后重新刷新页面，会发现页面的name会有一个稍微慢一点的页面加载过程，是从 {{name}} > 紫邪情，这里不方便演示，就不弄了。

而为了解决这个问题，可以使用v-cloak指令，本质是操作CSS罢了，
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117232034894-861099543.png" alt="image" />





> 小结：v-cloak指令

- 本质是一个特殊属性，Vue实例创建完毕并接管容器后，会删掉v-cloak属性。
- 使用CSS配合v-cloak指令可以解决网速慢时页面展示出 `{{xxx}}` 插值表达式  / axios交互延迟的问题。

axios就是ajax，Vue本身并不支持交互，所以才需要借助axios插件来进行交互，这也是玩了Vue之后必须掌握的一个知识点，对于学Java的人来说太简单了，上手的话，看一眼就懂了，那就是一个链式调用罢了，和StringBuilder一样链式调，只是需要传一些东西进去而已，而传的东西在ajax中都见过。

当然：只学了Vue，然后使用ajax一样可以进行数据发送，只是axios更轻小而已。





#### v-once指令

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117234025445-410778601.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117234035849-1663479424.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117234119334-2123289299.png" alt="image" />



因此：使用v-once实现效果。

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117234159933-605537265.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117234217613-194233763.png" alt="image" />







> 小结：v-once指令

- v-once所在节点在初次动态渲染后，就视为静态内容了。
- 以后数据的改变不会引起v-once所在结构的更新，可以用来做性能优化。
- 注意：和事件修饰符中的once区别开
  - 事件中的once修饰符是说的事件（ 函数 / js表达式）只执行一次。
  - 而v-once指令是说的对div容器中的模板只动态渲染一次。







#### v-pre指令
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117235608992-526325245.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220117235634345-161397174.png" alt="image" />







> 小结：v-pre

- v-pre是跳过其所在节点的编译过程，也就是：原本是什么样子，那么Vue就拿到的是什么样子，不会再去解析它。
- 可利用它跳过：没有使用指令语法、没有使用插值表达式的节点，从而加快编译，优化性能。



### 自定义指令

#### 函数式定义

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>函数式定义</title>

    <script src="../../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2>原来的数值为: {{number}}</h2>

        <hr/>

        <!-- 使用自定义指令 -->
        <h2> 扩大10倍之后的数值: <span v-big = "number"/> </h2>

        <hr/>

        <button @click = "number ++"> <h4>number+1</h4></button>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            number: 10
        },

        // 设置自定义指令 - 函数式
        // directives里面是可以写N多个自定义指令的，加了s嘛
        directives: {

            /* 
                element     是自定义指令所在结构的HTML标签元素（ 节点 ）
                binding     是模板中自定义指令和HTML元素进行的绑定的信息，在这个绑定中有一些需要的东西
                            注：此绑定和v-bind中的绑定不是一个概念
                           	   v-bind是指的HTML元素内的"属性"和值的绑定
            */
            big(element , binding){
                element.innerText = binding.value * 10
            }
        }
       });
    </script>
  </body>
</html>
```

![image-20240214221607151](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214221537336-2139420315.png)





> 注意点：上面自定义的指令什么时候会被调用？

1. 毫无疑问，HTML元素和自定义指令成功绑定时，自定义指令就会被调用，注意：这句话有坑，待会儿下一种自定义方式就会弄到这句话。

看到的效果就是初始化页面一上来就调用了一次，但是里面的门道不是这样的，成功绑定是什么意思，下一种自定义指令再说明。

2. 自定义指令所在的模板（ Div容器 ）被重新解析时会被调用。

别混淆了此种说法：自定义指令所依赖的数据发生改变时就会调用自定义指令，此种说法是错的。来个实例

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220118140139850-586534426.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220118140218020-609919652.png" alt="image" />





#### 对象式定义

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>对象式定义</title>

    <script src="../../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
      <!-- 现在做一件事情：模拟lable for标签效果 就是：光标位置锁定 别在意设计好不好-->
      <h2>计数: {{count}}</h2>
      <button @click = "count++">地址: </button>
      <input type="text" v-findfocus:value= "address">
      
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
          count: 1,
          address: '中国大陆'
        },

        directives: {
          // 先用函数式来玩一下 注意：自定义指令 多个单词别整驼峰命名啊，识别不了的
          findfocus( node, binding){
            console.log(node , binding);
            node.value = binding.value;
            // 获取焦点
            node.focus();
          }
        }
       });
    </script>
  </body>
</html>
```

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220118163613507-81621048.png" alt="image" s/>



现在再加一点要求：就是页面一初始化时，焦点就在input框中，此时发现貌似做不到（用js可以做到啊，只是现在是用Vue）。

为什么Vue不行？因为这要考虑到Vue的加载顺序问题（重点咯，开始衍生出Vue的生命周期和钩子函数咯，也就可以很容器理解前面使用v-cloak解决页面闪烁的问题了）。
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220118164925484-1896904902.png" alt="image" />



所以：应该把`focus()`获取焦点，放到页面渲染时再做，不然：无法聚焦。

相应的，用函数式并不能做到，因为：函数式直接一条龙在内存执行完了，所以：需要使用对象式自定义指令来解决这种小细节问题。
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220118170502689-1522353607.png" alt="image" />



`update()`中不写逻辑目前是不报错的，但是：最好还是写上。

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220118170602280-973393855.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220118170618940-1094106864.png" alt="image" />







#### 自定义指令的两大坑 和 补充

1. **命名的坑：** 这个坑在前面玩对象式时已经说过了，多个单词的名字别采用驼峰命名，但是：只说了这一点，并没有说正确形式应该怎么命名。

现在采用驼峰命名看一下这个坑：

<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220118174631094-1294214290.png" alt="image" />



<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220118174739410-1378313194.png" alt="image" />



> 多个单词组成的指令名正确形式，采用 - 分隔开，指令定义中回归js原始配置

  <img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220118175051180-1487426079.png" alt="image" />



  <img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220118175135182-695985940.png" alt="image" />

  

  

  

2.  **this指向的坑**

  <img src="https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214224037503-1228141863.png" alt="image" />





  ![image-20240214224136480](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240214224106123-40052284.png)



> 为什么自定义指令中的this是window对象？

这是因为：自定义指令已经不是Vue自己本身的东西了，而是根据需要自行设计，因此：此时你要操作的是HTML的DOM节点，因此：为了方便，Vue就把this还回来了，重新给了window。

**this变了，那想要拿到Vue中的数据咋办？**
<img src="https://img2020.cnblogs.com/blog/2421736/202201/2421736-20220118181528005-717127289.png" alt="image" />

  





3. **补充：自定义全局指令**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>全局指令配置</title>

    <script src="../../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2>计数: {{count}}</h2>
        <button @click = "count++">地址: </button>

        <!-- 正确形式命名 采用 - 分隔开 -->
        <input type="text" v-find-focus:value= "address">
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 定义全局指令 - 对象式
      /* 
            第一个参数  指令名
            第二个参数  配置项（ 就是把前面的对象配置{}这个花括号整体搬过来了而已 ），函数式也是一样的
                        只不过函数式需要使用 function给包裹起来而已
      */
      Vue.directive('find-focus',{
            // 1、HTML元素和指令进行绑定时执行的函数
            bind(node, binding){
              node.value = binding.value;
            },

            // 2、指令所在元素被插入被解析完毕，插入到页面后执行的函数
            inserted(node, binding){
              node.focus();
            },

            // 3、指令所在模板被重新解析时执行的函数
            update(node, binding){
              node.value = binding.value;
            }
        })

      /*   // 函数式全局指令
        Vue.directive('find-focus', function(){
            // 1、HTML元素和指令进行绑定时执行的函数
            bind(node, binding){
              node.value = binding.value;
            },

            // 2、指令所在元素被插入被解析完毕，插入到页面后执行的函数
            inserted(node, binding){
              node.focus();
            },

            // 3、指令所在模板被重新解析时执行的函数
            update(node, binding){
              node.value = binding.value;
            }
        })
 */
      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            count: 1,
            address: '中国大陆'
        },
       });
    </script>
  </body>
</html>
```



创建了全局指令之后，就可以摆脱只能在一个div容器中使用前面那种局部指令的缺陷了，也就是：现在可以再开一个div容器，然后在这个新开的div容器中一样可以使用全局配置中的东西。

全局配置东西基本上都是这么玩的，后续玩组件会用到。





#### 自定义指令总结

1. 局部指令

```javascript
对象式
new Vue({
	directives: {
		指令名: { 
			配置 
		}
	}
})



函数式
new Vue({
	directives: {
		指令名(){
			配置
		}
	}
})
```



2. 全局配置

```javascript
对象式
Vue.directive( '指令名',{
  	配置
})



函数式
Vue.directive('指令名', function(){
	配置
})
```



3. 配置中常用的3个回调

```javascript
bind( element, binding )			指令与元素成功绑定时调用

inserted( element, binding )		指令所在元素被解析完、插入到页面时调用

update（ element, binding )		   指令所在模板被重新解析时调用
```



4. 注意项：

- 1）、指令定义时不加 `v-` ，但使用时必须加 `v-`。
- 2）、指令名如果是多个单词，要使用 `-` 短横线分隔开，切忌使用驼峰命名（包括大驼峰和小驼峰）。







### Vue生命周期

#### 了解Vue生命周期的概念

做这么一个效果 —— 文本动态透明。

先用不对的方式来做一下：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>认识Vue生命周期概念</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2 :style="{opacity: num}">玩动态改变文本透明度</h2>
        {{ change() }}
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            num: 1,
        },

        // 改变透明度值大小
        methods: {
            change(){
                setInterval(() => {
                    if( this.num <= 0 ) this.num = 1;

                    this.num -= 0.01;
                }, 50);
            }
        }
       });
    </script>
  </body>
</html>
```



上面这样做之后，看起来好像成功了，但是运行一会就会发现：电脑的风扇会使劲儿转^ _ ^ 。原因如下：
<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220118203552772-19732433.png" alt="image" s/>



改造：使用Vue的钩子函数
<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220118204853354-543470071.png" alt="image" />



**注意：钩子函数执行的次数问题**
<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220118205037963-1393571370.png" alt="image" />

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220118205113720-761663219.png" alt="image" />



这就是前面说 `mounted()` 为什么是：Vue解析完了模板之后，准备把“初始的真实DOM”渲染到页面时会调用的一个函数。初始的真实DOM，是初次转成真实DOM时执行的，后面数值改变那是更新，对于模板来说，它不叫改变，别忘了前面玩diff算法时说过的流程 ———— 新的虚拟DOM会和初始虚拟DOM进行对比。





> 小结：Vue生命周期

- 定义：Vue在关键时刻帮我们调用的一些特殊名称的函数。
- 别名：生命周期回调函数 / 生命周期函数 / 生命周期钩子。
- 注意：
  - 生命周期函数的名字不可更改，但是：函数中的执行体是可以自己根据需求编写的。
  - 生命周期函数中的this指向是Vue实例 或 组件实例对象。



**生命周期过程**

- 首先这个知识点的原理图在官网中有。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119135745530-103142089.png" alt="image" />





- 对上图中的内容做补充。下图这个图是为了先了解大概，接下来会简单说明一下图中的内容。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119133045755-824545654.png" alt="image" />











#### Vue的挂载流程

> 所谓的挂载在前面引入生命周期概念时说过了，就是Vue在调`mounted()`时，换言之：就是前面图中直到`mounted()`时的步骤就是挂载流程，只是需要解读一下。@紫邪情



先来看一下这部分：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119142002300-1524247555.png" alt="image" />



说在beforeCreate时还不可以使用vm访问data中的数据、methods中的方法，在这之前只开始初始化生命周期、事件，那就来测试一下：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Vue的挂载</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <h2>{{name}}</h2>
        <button @click = "print">切换名字</button>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{
            name: '紫邪情'
        },

        methods: {
            print(){
                console.log("正在执行print()函数");
            }
        },

        // 之前只开始初始化生命周期、事件，此时：还不能使用vm访问data和methods
        beforeCreate () {
            // 操作data中的数据 methods也是差不多的
            console.log( this.name );

            // 查看Vue内容，看看是哪些东西
            console.log(this);

            // 打断点  和java中的debug是一回事
            debugger;
        }
       });
    </script>
  </body>
</html>
```

运行程序，调出控制台，然后刷新页面进入debug断点：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119144117718-901257056.png" alt="image" />



接着来看这一部分：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119144323863-1390794462.png" alt="image" />



说的是：已经开始了数据监测、数据代理，可以使用vm访问data和methods了，那久测试一下。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119144743786-2086770627.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119145401327-1116604649.png" alt="image" />





接着看这一部分：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119145558511-2035888178.png" alt="image" />



先看下面部分，说：页面呈现的是未经编译的DOM结果且对DOM的操作，“最终”都不奏效，测试一下。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119150526010-33636766.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119150453794-1414104218.png" alt="image" />



同时还说：所有对DOM结构的操作都不奏效，那就试一下。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119150958582-7137341.png" alt="image" st />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119151119622-21679891.png" alt="image" s/>



现在接着回去看那个逻辑判断里面的内容：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119151934905-1520958385.png" alt="image" />



外层和内层就是下面这个：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119152313500-2090594963.png" alt="image" />



而所谓的template模板更简单：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119152814941-2051606924.png" alt="image" />



截图中有个地方容易产生歧义：template中应该还需要使用一个div给套起来，template中只能有一个根节点，而例子中有h2 和 button两个节点，会报错的，所以需要使用div（ 或其他标签）把要写的代码给套起来，可以对照Vue实例和容器的对应关系（ 一 一对应）。



最后来看这一部分：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119153027769-1300395734.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119154214489-262318439.png" alt="image"  />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119154411683-1782566606.png" alt="image" />



但是：这里有一个点需要注意

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119160716365-1768864344.png" alt="image" />



这里是将真实DOM使用”vm.$el“把真实DOM中的内容给复制了一份保存起来了，这一步很关键，为的就是后续 新的虚拟DOM 和 初始的虚拟DOM进行对比算法时，有相同的数据内容，那么新的虚拟DOM就会直接用初始虚拟DOM的数据（前面玩v-for中key的原理时，key值为index和数据id的区别），而要用初始DOM就需要找到初始DOM呗，它保存起来之后就方便找到了嘛。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119161754589-482583734.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119161832558-995163989.png" alt="image" />











#### Vue的更新流程

内容就一丢丢，下图中红色框起来的内容：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119162606893-1339277210.png" alt="image" />



来看`beforeUpdate()`

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119163325164-444782214.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119163215076-719750883.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119163528290-1137405702.png" alt="image" />





接着来看Updated

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119163711068-744260567.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119163742413-654105579.png" alt="image" />







#### Vue的销毁流程

内容就是图中的最后一点东西，**但是：这里面坑也是有点多的**

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119170726773-2047059685.png" alt="image" />



图中提到了 `vm.$destroy()` 和 销毁哪些东西，所以先看官网对销毁的介绍：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119170321076-217357837.png" alt="image" />



现在就用代码去验证一下图中的内容，需要改造一下代码：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119172208888-605065597.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119172254219-1751072276.png" alt="image" />





执行效果如下：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119172838789-1632407897.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119173153603-1931479316.png" alt="image" />



**开始踩另一个坑**

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119173349243-1760857341.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119173540667-1142898863.png" alt="image" />



**vm确实是销毁了，但是这个坑是前面说的销毁事件监听，它销毁的是：自定义事件（后续接触），所以Vue自身的事件还在，也执行了**

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119173946162-42297008.png" alt="image" s/>



**那又有问题：页面为什么没 +1 ？**

- 是因为`beforeDestroy ()`是一个贼尴尬的阶段，这里面确实可以拿到data、methods、指令这些；
- 但是：当一点击销毁（调用了 `vm.$destroy()` ），程序就会执行`beforeDestroy ()`，这里面的调用methods也会去执行，可是：关键点就是它不会再去调用` beforeUpdate ()` 和 `updated ()` ———— 正常流程是数据发生改变就会调这两个，但是：`deforeDestroy()`就不会调用，所以：才在前面说 此阶段一般干的事情是：关闭定时器、解除自定义事件绑定等操作。







#### Vue生命周期总结

Vue生命周期共有8个（4对 ），另外其实还有3个周期（ 但是：这三个需要到router路由中玩切换时才能整）。

```javascript
beforeCreate()		created()

beforeMount()		mounted()

beforeUpdate()		Updated()

beforeDestroy()		Destroyed()
```

其中：常用的钩子函数就两个

```javascript
mounted()				用来发送ajax请求、启动定时器、绑定自定义事件、订阅消息（后续说明）等	即：完成的是"初始化操作"

beforeDestroy()			用来清除定时器、解绑自定义事件、取消订阅消息等，即：做的是“收尾操作”
```



关于销毁vm的问题

- 销毁后借助Vue开发者工具是看不到任何信息的。
- 销毁后自定义事件会失效，但是：Vue原生DOM事件依然有效。
- 一般不会在`beforeDestroy()`中操作数据，因为就算操作了，那也不会触发`beforeUpdate()`  和 `updated()`。







## 组件化开发



### 组件概念

组件，对于学Java的人来说的话，这个词所要表达的意思再熟悉不过了。

**所谓组件就是：面向对象中的抽象、封装思想**；而所谓的**组件化就是：把功能用多组件的方式搭配起来编写**。

- 有一个根组件，旗下有N多微型组件 ，粗暴理解就是：SpringCloud中的`main()`方法可以搭配很多不同功能的注解，`main()`方法就是根组件，不同功能的注解就是微型组件，那这些功能组成的应用程序就是一个组件化应用，因此：这样做之后，好处就是利于维护和提高代码的复用性了。



但是对于前端的人来说，这个东西就需要特别解释一下，直接下定义就是：  **实现应用中局部功能代码和资源的集合**。

瞄一下官网，它里面有一个图：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220120120217690-1152831316.png" alt="image" />



所以：现在就可以理解前面下的定义为什么是局部功能代码和资源的集合了，局部功能就是某一个模块，是针对这个模块内部的，而这个模块内部的编写不就是CSS、HTML片段、JS代码吗，同时png、mp3等等这些就是资源咯。



**至于为什么要学组件化开发？**

- 一是：因为做的应用页面都是很复杂的，如果使用传统的三剑客CSS+HTML+JS，那么就会出现很多的js文件，不利于维护 和 编写很费力的。
- 二是：因为组件化开发可以极好的复用代码、简化项目代码、所以也就提高了运行效率。



**同时组件又有单文件组件（ 真实开发玩的）和 非单文件组件**

- **单文件组件：**就是指只有一个组件组成（ 即：是一个.vue的文件）。
- **非单文件组件：**就是有N多个组件组成。





### 非单文件组件

#### 使用组件

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>玩一下组件</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
      <!-- 3、使用组件 -->
      <person></person>
        
      <hr/>

      <hobbys></hobbys>

    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 玩组件三板斧
      // 1、创建组件
      const person = Vue.extend({
        // 这里在基础篇中怎么玩就怎么玩，相应的也有watch、computed.....
        // 但是：切记：不可以用el和对象式data（必须是函数式）
        /* 
          不可以用el的原因是：el指向的是具体的容器，这是根组件做的事情，现在这是小弟
          不可以用data对象式，而必须用函数式：是因为对象是一个引用地址嘛（玩Java的人很熟悉这个对象的事情）
                                 如果用引用一是Vue直接不编译、报错，二是就算可以用对象式，那几个变量都可以
                                 指向同一个对象，那么就会产生：一个变量修改了对象中的东西，那么另一个变量指向
                                 的是同一个对象，因此：数据也会发生改变
                             而函数式则不会，因为：函数式就会是哪个变量用的，里面的return返回值就是属于哪个变量
        */

        // 使用模板，这个就需要直接写在组件里面了，如果：放到div容器的模板中，是会报错的
        template: `
          <div>
            <h2>{{name}}</h2>
            <h2>{{age}}</h2>
            <h2>{{sex}}</h2>  
          </div>
        `,
        // 切记：这里是使用data的另一种写法 —— 函数式，必须用，前面基础篇说过了
        data(){
          return {
            name: '紫邪情',
            age: 18,
            sex: '女'
          }
        }
      })

      // 再创建一个组件
      const hobbys = Vue.extend({
        template: `
          <div>
            <h2>{{one}}</h2>
            <h2>{{two}}</h2>
            <h2>{{three}}</h2>  
          </div>
        `,
        data(){
          return {
            one: '抠脚',
            two: '玩',
            three: '酒吧'
          }
        }
      })

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        // 这里面也可以使用这个编写data，和以前一样
        data:{},

        // 2、注册组件
        components: {
          // 前为 正式在页面中用的组件名（ div模板中用的名字 ）  后为组件所在位置
          // person: person,           // 这种同名的就可以简写

          // 简写
          person,
          hobbys
        }
       });
    </script>
  </body>
</html>
```

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220120225806501-1073652484.png" alt="image" />





> 小结：使用组件

1. Vue中使用组件的三板斧
   - 创建组件；
   - 注册组件；
   - 使用组件（写组件标签即可）。



2. 如何定义一个组件？
   
   - 使用`Vue.extend( { options }` )创建，其中options 和 `new Vue( { options } )` 时传入的哪些option“几乎一样”，区别就是：
   
     - 1）、el不要写 ——— 因为最终所有的组件都要经过一个vm的管理（ 根组件），由vm中的el决定服务哪个容器。
   
     - 2）、data必须写成函数式 ———— 因为可以避免组件被复用时，数据存在引用关系。
   
   - 另外：template选项可以配置组件结构。



3. 如何注册组件？

   - 1）、局部注册： 靠new Vue的时候传入components选项。

   - 2）、全局注册：靠 `Vue.component('组件名' , 组件)`。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220120231514412-750022953.png" alt="image" />



- 真实开发中一般都是用前面玩的局部组件，局部变全局都是一样的套路，去掉s，然后使用Vue来调，最后加入相应的名字和配置即可。

- 编写组件标签。

```html
<person></person>
```



#### 使用组件的注意点

1. **创建组件时的简写问题**

```javascript
// 1、创建局部组件( 完整写法 )
const person = Vue.extend({
    template: `
      <div>
          <h2>{{name}}</h2>    
          <h2>{{age}}</h2>
      </div>
    `,
    data(){
        return {
            name: '紫邪情',
            age: '女'
        }
    }
})

// 简写形式
const person2 = {
  template: `
      <div>
          <h2>{{name}}</h2>    
          <h2>{{age}}</h2>
      </div>
    `,
  data(){
        return {
            name: '紫邪情',
            age: '女'
        }
    }
}
```

上面两种都可以。但是：简写形式在底层其实也调用了 `Vue.extend()`

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220121154253541-2078088171.png" alt="image" />



![image-20240215000654869](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215000625219-1905662358.png)

验证完了，记得把源码的断点去掉。







2. **组件名的问题**

- 1）、**组件名为一个单词时，使用全小写字母 / 首字母大写都没问题**。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220121154927392-1387731804.png" alt="image" />



![image-20240215000823783](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215000754443-1459368936.png)





- 2）、**组件名为多个单词组成时，全部用小写  或 使用 `-` 进行分割都没问题**。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220121155704043-33456825.png" alt="image" s/>



![image-20240215000924692](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215000854584-448406154.png)



还有一种就是上图中的效果：驼峰命名

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220121160632609-67278161.png" alt="image" style="zoom:50%;" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220121160715764-1078760791.png" alt="image" style="zoom:50%;" />



但是：这种驼峰命名需要注意，有些人就不可以。因为：严格来说这种命名是后续的技术使用脚手架玩时的方式，但是：有些人就可以，比如我上图中的效果，因为这是Vue版本的问题，要看源码的话，从下图这里往后看即可。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220121161752001-541215883.png" alt="image" />



- 3）、注意组件名别和HTML中的原生标签名一致，会冲突报错，HTML的限制就是上图中看源码中的哪些，如果非要用HTML标签名，让人见名知意，那就在原生HTML标签名前加一些特定的词，如;my-input这种，千万别用：input、h2....此类名字来命名组件名。





3. **关于组件在使用时的注意项**

- 1）、可以使用双标签，如：`<my-info></my-info>`，这种肯定没任何问题。

- 2）、也可以使用自闭合标签，如：`<my-info/>`，但是这种有坑，这种使用方式需要脚手架支持，否则数据渲染会出问题。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220121162701519-1847482790.png" alt="image" />



![image-20240215001136561](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215001106570-771102568.png)









#### 使用组件注意点总结

1. **关于组件名**

- 一个单词组成时：
  - 1）、全小写，如：person。
  - 2）、首字母大写，如：Person。
- 多个单词组成时：
  - 1）、全小写，如：myinfo。
  - 2）、使用 `-` 分割，如：my-info。
  - 3）、驼峰命名，如：MyInfo，但注意：目前没用脚手架之前最好别用，是因为指不定一会好使，一会不好使。
- 注意事项：
  - 1）、组件名最好别和HTML的标签名一致，从而造成冲突（非要用，可以采用加词 或 使用 `-` 分割）。
  - 2）、可以在创建组件时，在里面配置name选项，从而指定组件在Vue开发者工具中呈现的名字。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220121163732203-293464250.png" alt="image"  />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220121163832192-903888280.png" alt="image" />

> 此种形式在第三方组件时会见到





2. **关于组件标签（使用组件）**

- 1）、使用双闭合标签也行，如：`<my-info></my-info>。`
- 2）、使用自闭合标签也行，如：`<my-info/>` 。但是：此种方式目前有坑，会出现后续组件不能渲染的问题，所以需要等到后续使用脚手架时才可以。



3. **创建组件的简写形式**

```javascript
const person = Vue.extend({
    配置选项
})

可以简写为：const person = { 配置选项 }
```





#### 组件的嵌套

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>组件嵌套</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app">
        <!-- 3、使用组件 -->
        <info></info>
    </div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 1、定义组件
      const person = {
          template: `
            <div>
                <h2>{{name}}</h2>    
                <h2>{{sex}}</h2>
            </div>
          `,
          data(){
              return {
                  name: '紫邪情',
                  sex: '女'
              }
          }
      }

      const info = Vue.extend({
          template: `
            <div>
                <h2>{{address}}</h2>    
                <h2>{{job}}</h2>
                <!-- 这个组件中使用被嵌套的组件 -->
                <person></person>
            </div>
          `,
          data(){
              return {
                  address: '浙江杭州',
                  job: 'java'
              }
          },

          // 基础组件嵌套 —— 这个组件中嵌套person组件
          /* 
            注意前提：被嵌套的组件 需要比 当前嵌套组件先定义（ 如：person组件是在info组件前面定义的 ）
                     原因：因为Vue解析模板时，会按照代码顺序解析，如果定义顺序反了
                           就会出现：这里用到的组件 在 解析时由于在后面还未解析从而出现找不到
          */
         components: {
             person,
         }
      })

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{},

        // 2、注册组件 —— 由于info组件中 嵌套了 person组件，所以在这里只需要注册 info组件即可
        components: {
            info,
        }
       });
    </script>
  </body>
</html>
```

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220121215640110-1765447723.png" alt="image" />





> 另一种嵌套：开发中玩的，我对那个div容器起的id值为app，是有用的

在开发中的嵌套是一个vm管理独一无二的app（就是application 应用程序的意思），然后由app管理众多小弟：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220121221514791-383429444.png" alt="image" />



所以，现在来玩一下这种组件嵌套

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>vm管app，app管众多组件</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app"></div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;


      // 1、定义组件
      const person = {
          template: `
            <div>
                <h2>{{name}}</h2>    
                <h2>{{sex}}</h2>
            </div>
          `,
          data(){
              return {
                  name: '紫邪情',
                  sex: '女'
              }
          }
      }

      const info = Vue.extend({
          template: `
            <div>
                <h2>{{address}}</h2>    
                <h2>{{job}}</h2>
                <!-- 这个组件中使用被嵌套的组件 -->
                <person></person>
            </div>
          `,
          data(){
              return {
                  address: '浙江杭州',
                  job: 'java'
              }
          },
         components: {
             person,
         }
      })

      // 再定义一个app组件，用来管理其他组件
      const app = {
          // 这个app组件没有其他的东西，就是注册和使用被管理组件而已
          components: {
              // 有其他组件也可以注册在这里面，这里由于info管理了person，所以只注册info即可
              info
          },
          template: `
            <div>
                <info></info>   
            </div>
          `,
      }

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{},

        // 由于组件被app管理，所以：只注册app组件即可
        components: { app },

        // 使用组件
        template: `
            <div>
                <app></app> 
            </div>
        `,
       });
    </script>
  </body>
</html>
```

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220121221241435-1900676993.png" alt="image" />





#### 认识VueComponent()函数

1. **来看一下组件到底是谁？**

基础代码：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>认识VueComponent</title>

    <script src="../../js/vue.js"></script>
  </head>

  <body>
    <!-- 被 vm 实例所控制的区域 -->
    <div id="app"></div>

    <script>
       // 去除浏览器控制台中的警告提示信息
      Vue.config.productionTip = false;

      // 1、定义组件
      const person = Vue.extend({
          template: `
            <div>
                <h2>{{name}}</h2>    
                <h2>{{job}}</h2>
                <h2>{{address}}</h2>
            </div>
          `,
          data(){
              return {
                  name: '紫邪情',
                  job: 'java',
                  address: '浙江杭州'
              }
          }
      })

      const app = {
          components: {person},
          template: `
            <div>
                <person></person>    
            </div>
          `,

      }

      // 创建 vm 实例对象
      const vm = new Vue({
        // 指定控制的区域
        el:'#app',
        data:{},
        components: {app},
        template: `
            <div>
                <app></app>    
            </div>
        `,
       });
    </script>
  </body>
</html>
```

现在就来见一下组件的真身（在前面玩this的时候说过，this指向的是：Vue实例 / 组件实例对象），因此：用this就可以知道组件的真身。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220122152200147-1501001304.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220122152322794-1312940761.png" alt="image" />



既然知道了组件真身是`VueComponent()`，那么先去源码中看一下它：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220122152642129-165711524.png" alt="image" />



源码提取出来就是下面的样子：

```javascript
var Sub = function VueComponent (options) {
        this._init(options);  <!--里面的重要逻辑封装在了_init()中了，目前不要去看-->
      };
  
      return Sub
};
```

**经过前面的分析和查看源码得出两个结论：**

1. 所有的组件指的都是`VueComponent()`。
2. 每一个组件都调用了`VueComponent()`，但它们都不一样（源码中有嘛，每次都是创建了一个全新的sub，sub就是`VueComponent()`，最后把这个sub返回去了，验证一下咯。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220122154717139-2147436929.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220122154819826-759080284.png" alt="image" />



但是：这里有一个有意思的东西，就是第一条说的组件就是指`VueComponent()`，从而会出现不了解的人认为：每个组件都是调了同一个`VueComponent()`，来看一下：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220122155255489-880385657.png" alt="image" />



这两个长得一模一样，所以就会让人误会，但这是一个构造函数。

构造函数，那就是Vue每次解析模板时（div容器使用的组件标签），就会去帮忙创建对应的组件，调用了构造函数。

怎么调用的？new出来的嘛，所以：这两个组件对象肯定不一样（前面先验证是否一样就是为了注意这点，看起来一样，实质不一样 ，两个组件创建的是不同的`VueComponent()` ）。

**同时上面说到，Vue解析模板时，会帮忙去创建`VueComponent()`，那么是谁去帮忙创建的？**

- 答案就是创建组件时，里面的`Vue.extend()`去帮忙创建的，这不是我们程序员自己整出来的，看一下源码：




<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220122160042483-504526350.png" alt="image" />





> 小结：VueComponent()

- 组件本质是一个名为`VueComponent()`的构造函数，且不是程序员自己定义的，是`Vue.extend()`生成的。
- 我们只需要写组件标签（如 `<person></person>` 或 `<person/>` ），Vue解析时会帮我们创建组件的实例对象，即：Vue帮我们执行了`new VueComponent( { options配置选项 } )`
  - 注意点：每次调用`Vue.extend()`，返回的都是一个全新的VueComponent



- 关于this的指向问题
  - 1）、在组件配置中：data函数、methods函数、watch中的函数、computed中的函数，它们的this均是【VueComponent实例对象】。
  - 2）、在 `new Vue()` 配置中：data函数、methods函数、watch中的函数、computed中的函数，它们的this均是【vue实例对象，即：前面玩的vm 】。
- VueComponent实例对象，简称：vc（或：组件实例对象）；Vue实例对象，简称：vm。



但是：vm和vc也有一个坑



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220122214949207-1560586309.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220122215202295-1679001352.png" alt="image" />



观察结构会发现：vm和vc如出一辙，什么数据代理、数据监测等等，vm有的，vc都有，所以vm中的配置项在vc中都可以配置，但是：**vm和vc不能画等号，它们两个不一样**

- vm是Vue实例对象，vc是组件实例对象
- vm中可以使用el选项，而vc中不可以（只要用el就报错）
- 在vm中，data可以用函数式和对象式，但是在vc中data只能用函数式
- vm是大哥，vc是小弟，vc是vm的组件，或者直接说：组件实例对象vc是小型的Vue实例对象
- vm和vc之间很多东西只是复用了而已。

这里说的复用，里面有大门道，vm和vc之间是有关系的，这里需要原型对象知识

1）、函数肯定有ProtoType（显示原型属性）；

2）、而实例（如：`const person = vue.extend()`中的person）肯定有 `_ _proto _ _` （隐式原型属性）；

而这二者指向的是同一个对象：即，该实例的原型对象，而vc和vm之间就是通过这二者关联起来的。



**vm和vc之间的内置关系是：`VueComponent.prototype._ _proto _ _ === Vue.prototype`**

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220122232340126-1982785462.png" alt="image" />



图中：VueComponent的原型对象通过 `_ _proto _ _`理论上应该直接指向object的原型对象，但是：Vue做了巧妙的事情：就是让VueComponent的原型对象通过 `_ _proto _ _`指向了Vue的原型对象，这样做的好处就是：**让组件实例对象 可以访问到 Vue原型上的属性、方法**。



以上的内容就属于非单文件组件相关的，接下来就看单文件组件，也是开发中会做的事情。



### 单文件组件

单文件组件：就是只有一个文件嘛，`xxxx.vue`。

而xxxx就是前面说过的组件命名：
- 单个单词：全小写、首字母大写。
- 多个单词：用 `-` 进行分割、大驼峰命名。
- 而开发中最常用的就是：首字母大写和大驼峰命名。



#### 疏通单文件组件的编写流程

前提：如果自己的编辑器是vscode，那么就给编辑器安装vetur插件（用其他的也行），然后重启vscode，这个插件就是为了能够识别`xxxx.vue`文件的；如果自己是用的IDEA编辑器来写的vue，那么安装了vue.js插件就可以了。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123140501758-678436950.png" alt="image" />



##### 创建xxxx.vue文件

这个创建的就是单文件组件，前面玩非单文件组件，不是有三板斧吗，对照来看。

创建了`xxx.vue`之后，是一个空文件，里面要写的东西就三样（模板template【即html+vue】、交互script、样式style），里面内容也对照非单文件组件来看。

```html
<template>
  <div class="temp">
      <!-- 这里面就是模板 以前在非单文件组件中用的template选项是怎么写的，这里面就是怎么写的-->
      <h2>{{name}}</h2>
  </div>
</template>


<script>
    // 这里面就是交互（ data、methods、watch、computed..... ）
    // 就是非单文件组件中的定义组件
/*    const person = vue.extend({
        // 这里就最好配置name选项了，一般都是当前创建的xxxx.vue中的xxxx名字即可
        name: 'Person',
        data() {
            return {
                name: '紫邪情'
            }
        },
        // 这里面还可以写什么methods、watch.....之类的
    })
*/

    // 但是：上面是对照非单文件组件来写的，在这个单文件中其实换了一下下
    // 1、这个组件是可以在其他地方复用的，所以：需要把这个组件暴露出去，然后在需要的地方引入即可
    /* 
        这里需要使用到js中模块化的知识
        export暴露 import引入嘛
        但是：export暴露有三种方式
            1、分别暴露  export const person = vue.extend({ 配置选项 })，
                就是在前面加一个export而已
                可是：根据前面非单文件的知识来看，这个是可以进行简写的
                export person {}
            2、统一暴露 就是单独弄一行代码，然后使用 export { 要进行暴露的名字 }，多个使用 , 逗号隔开即可
            3、默认暴露（ vue中采用的一种，因为引入时简单 ）  export default 组件名{ 配置选项 }
                但是：组件名就是当前整个文件，所以可以省略
                默认暴露引入： import 起个名字 from 它在哪里
                而其他的暴露方式在引入时会有点麻烦
    */

   // 正宗玩法
   export default {
       name: 'Person',
       data() {
           return {
               name: '紫邪情'
           }
       },

       // 再配置其他需要的东西也行 如：methods、watch、computed.....
   }
</script>



<style>
	/* 这里面就是template中的样式编写, 有就写，没有就不写 */
    .temp{
        color: purple;
    }
</style>
```

> xxx.vue文件，如果使用的是vscode+vetur，那么上面的模板可以用快捷键 ：就是输入 `<v`  然后回车就可以生成了（也可以百度“vue生成模板”进行快捷键配置）。







##### 注册组件到app中

前面玩过，vm管app，app管其他的小弟，所以需要一个app组件，创建一个`app.vue`：

```html
<template>
  <div>
      <!-- 3、使用app管理的组件 -->
      <person></person>
  </div>
</template>

<script>
    // 1、引入定义的person组件(要是有其他组件要引入是一样的套路)
    // 1Person.vue这个名字不正规啊，我只是为了排序才加了一个1
    import person from "./1Person.vue"  

    export default {
        name: 'App'
        // 2、注册引入的组件
        components: {person}  // 完成了引入和注册之后，在这里面就可以用引入的组件了
    }
</script>

<style>
	/* app是为了管理其他所有的组件，所以这个style其实不写也行（ 按需要来吧 ） */
</style>
```







##### 将app和vm绑定起来

新建一个`main.js`文件，创建这个文件的目的：一是让app和vm绑定，二是浏览器并不能识别`.vue`文件，所以根本展示不了，因此：需要将`.vue`文件转成js文件，这样浏览器就能解析了。

```javascript
// 1、引入app组件
import App from "./2App.vue"

// 2、把app组件和vm进行绑定
new Vue({
    // 这里面和以前一样写法，当然：这里的el值绑定的是容器id，怕误会改成root也行
    el: '#App',
    components: {App}
})
```



##### 创建容器

前面app组件和vm绑定了，但是vm中指定的el值，它绑定的容器还没有啊，因此：创建`index.html`。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>创建el容器</title>

    <!-- 
        记得要引入js，而此时就需要引入两个js，一个是main.js，一个是vue.js 
        可是：在解析下面的容器时，可能会导致js渲染不及时出问题
        因此：引入js最好放在下面容器的后面引入
    -->
</head>
<body>

    <div id="App">
        <!-- 
            2、使用app组件，可以在这里使用，也可以不在这里使用直接在app.vue中使用template选项进行使用
        -->
        <App></App>
    </div>
    

    <!-- 
        1、引入js 
            vue.js是因为：main.js中new vue()需要它，所以：先引入vue.js
            其次再引入main.js
    -->
    <script src="../../../js/vue.js"></script>
    <script src="./3Main.js"></script>
</body>
</html>
```

经过上面的操作之后，玩Vue单文件组件的流程也就完了，整体结构就是如下所示：

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123161054714-740929296.png)





而整个流程按照解析的逻辑来看就是如下流程：

1. 进入`index.html`，创建了`div id = "App"`容器，然后引入`vue.js`，再引入`main.js`



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123161409322-1213574856.png" alt="image" />



2. 但是：引入`main.js`，去`main.js`里面开始解析时，发现：需要引入`App.vue`，所以：接着引入`App.vue`



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123161552112-2142762492.png" alt="image" />



3. 进入`App.vue`，又发现需要引入`Person.vue`



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123161700474-501141588.png" alt="image" />



4. 将所有东西都引入完了之后，就可以依次进行渲染了（逻辑就不说明了），而经过上面的逻辑梳理之后会发现：**`main.js`就是入口，是从`main.js`开始引入，从而把其他的东西也给引入进来了**。





当然：以上的东西弄完之后，还启动不了，一启动就会报错。这是因为：浏览器不能解析ES6语法，这需要使用另外一个技术，脚手架来支持

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123162126603-891348612.png" alt="image" />









### 认识脚手架 vue cli

cli全称： command line interface ，即：命令行接口工具，但是：一般说的都是脚手架，正规名字说起来太官方、绕口。

在vue官网有这个脚手架生态：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123165415132-1175644696.png" alt="image" />







#### 使用nodejs配置脚手架

> ==前提：保证自己的电脑有nodejs==。



##### 安装NodeJS

nodejs的配置很简单，[官网](https://nodejs.org/en/download/) 进行下载（版本最好选择大于等于16的，后续学其他东西时方便）、一直next、最后修改环境变量。LTS就是稳定版，而CURRENT就是更新版（新特性就丢在这里面的，可能会出现bug，所以不推荐下载）

> 有个注意点：选择安装目录时，别把nodejs安装到系统C盘了，不然很大可能出现权限不足、无法操作的问题，特别是：如果自己的电脑没升级，还是家庭版的而不是专业版的，这种问题更常见（当然话说回来，事非绝对，有些人就是不会出问题，所以其实可以先安装C盘试一下，遇到了就当碰壁学经验了）。

- 出现这种问题就需要切换到管理员身份运行cmd才可以进行安装vue-cli了，甚至有时会奇葩点：需要在管理员身份下运行`npm clean cache –force。`。
- 然后再进入到 C:/users/appdata/roaming 下把一个叫做nom-cache的缓存文件删了，最后再用管理员身份运行`npm clean cache –force`清除缓存，搞完这些才可以安装vue cli脚手架。



安装成功之后是如下效果

![image-20240215101843369](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215101813843-861386900.png)



查看一下是否成功：进入DOS窗口（win+r，输入cmd回车），以下内容表明成功

<img src="https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230816013905231-1921984768.png" alt="image" />



但是：现在npm的配置和缓存文件都在 C:/users/appdata/roaming/npm 和 appdata/local/npm-cache 中

<img src="https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230816014021609-1894023860.png" alt="image" />



因此：我们需要去改动这两个地方（知道了这两个目录，不用改也可以，后面什么事都可以不做了，对后续的操作没影响的 ，嫌麻烦就可以改）



##### 配置NodeJS

1. 在安装的nodejs中新建 `node_global `和 `node_cache` 两个文件夹（前为全局配置路径，后为npm缓存路径）。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124204421306-719446743.png" alt="image" style="zoom:80%;" />



2. 用管理员权限打开DOS窗口，执行如下命令（路径记得复制成自己的）。

```bash
全局设置
C:\WINDOWS\system32>npm config set prefix "D:\install\Nodejs\node_global"

C:\WINDOWS\system32>npm config set cache "D:\install\Nodejs\node_cache"


检查是否成功
C:\WINDOWS\system32>npm config get prefix
D:\install\Nodejs\node_global

C:\WINDOWS\system32>npm config get cache
D:\install\Nodejs\node_cache
```

可见成功修改，但是：还需要做最后一步，去改环境变量。

在改环境变量之前，在刚刚新建的 `node_global `目录下，再新建一个 `node_modules` 文件夹

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124210635877-365488382.png" alt="image" style="zoom:80%;" />



3. 修改环境变量



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124210530991-2024766786.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124210844612-739405396.png" alt="image" />



最后一路点OK即可，测试是否成功，可以选择安装一个vue包来测试

```bash
npm i -g vue
```

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124234248888-1021672162.png" alt="image" />

此时可能出现报一堆的ERROR，最后一行的大概意思就是让使用 root/ admin...用户（也就是让用管理员运行dos窗口，再执行命令）。



##### 报一堆ERROR错误的解决办法

此时：做一个操作即可，回到NodeJS安装的根目录

![image-20240215102744577](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215102715047-195517840.png)



右键选择属性、安全、高级

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124234130066-2124034218.png" alt="image" />



当然：要是自己的电脑在这个安全界面中，直接编辑权限，然后把“写入权限、修改权限”√上、应用确定是可以的，那就可以了，要是不行就接着往后看。

![image-20240215103015644](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215102945662-25757104.png)



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124235340948-1947998348.png" alt="image" />



然后再使用`npm install -g xxx`就可以了（前面安装vue包报错的，以防万一先去前面配置的npm_cache中将不完整的vue文件夹删掉【shift+del彻底删除】，再执行此命令）。



安装之后，在刚刚新建的 node_global 和 node_cache 中是有东西的：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124211415181-31111214.png" alt="image" s/>



> 如果想要把全局配置恢复为初始化配置的话，也很简单，系统`C:/users/.npmrc`的文件，删了就可以了。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125120314537-124024968.png" alt="image" />





##### 配置npm镜像

配置成功了nodejs之后，就可以使用npm指令了。

但是：npm是国外的，我们拉取东西时就犹如隔了一道墙，很慢。

因此：拉取淘宝的镜像，从而使用cnpm来代替npm指令，拉取淘宝镜像链接：`npm config set registry https://registry.npmmirror.com`。别用  `npm config set registry http://registry.npm.taobao.org` （2022 年 5 月 31 日 停止服务）。

拉取镜像这里开始就一定要保证自己的网络流畅，不然很容易导致一是淘宝镜像拉取失败（看起来成功了，但是一用就报cnpm不是内部命名 ，这种情况要么权限不够，需要管理员身份打开dos窗口；要么cnpm没拉完整），二是后面安装脚手架时，要是网络不好，也很容易出现看起来成功了，但是：一用就发现vue不是内部指令。



下图是我重新拉取一遍的效果：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123201331716-1148811536.png" alt="image" />





##### 脚手架创建Vue2项目

1. **全局安装vue cli**。指令： `npm install -g @vue/cli`。

- `npm` 是nodejs的指令 拉取了淘宝镜像之后，就可以使用cnpm代替了 。
- `install` 是安装的意思，可以简写为 i。
- `-g` 是全局安装。
- `@vue/cli`  是安装的东西。

> 有个注意点：要是有人知道可以使用 `npm install -g vue-cli` 这样安装脚手架的话，可以用，没错的，但是：目前别这么安装。它安装的脚手架是2.x的，用这种方式安装的不能保证vue（目前版本是1 - 3）和vue-cli（目前版本是1 - 4 ）的版本很适合，所以后续使用一些命令时可能会出现版本不足的问题，让把版本升级，而使用`@vue/cli`安装的是最新版本。



2. **创建一个文件夹，并进入目录中。** 使用指令：`vue create xxxx`

- `vue create`  是cli 3.x的命令，要是前面安装脚手架时是乱整的，就会发现：这个命令用不了，要是出现这样的话，那么执行一遍这个命令，会提示你：卸载以前的cli，然后执行什么命令安装cli。
- `xxx` 就是要创建的项目名。



![image](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215104608578-734698425.png)

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124115640959-1213435605.png" alt="image"  />

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124120847991-770527317.png" alt="image"/>

出现如上图就说明在开始拉取依赖了。



3. **启动项目**

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124121652987-1193057104.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124123522782-524045639.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124123654310-967872506.png" alt="image" />



想要退出启动的程序，一是直接点窗口右上角的×，二是按两次ctrl+c即可。



> 补充：以下内容是以前使用` vue init webpack xxx` 来安装的vue-cli。

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123224456196-511773944.png)



查看一下自己的vue-cli安装成功没有，指令： `vue list`

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123224709173-1717680453.png)

vue-cli安装成功之后，想找它就在 C/user/appdata/roaming/npm/node_modules 中可以看到（前面配置了环境变量就在配置的路径中）。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123233314203-131102479.png" alt="image" />



当然：有可能有些不听劝的人在安装nodejs时，搞了一些不必要的操作，导致有些文件需要管理员权限才可以。

因此：就会出现这里安装vue-cli时失败，报的错就是什么node_glob.....什么爪子之类的，总之：就是权限不够，然后使用管理员身份做前面的操作就发现突然吃鸡了，这就是典型的权限不够导致的。这样的话，以后：你使用vue相关的东西时，就都得使用管理员权限才可以进行操作。







2. 创建一个文件夹，然后进入文件夹，使用指令：`vue init webpack xxxx`，进去之后会做一些操作。

- `init` 是初始化。
- `webpack` 就是骨架，就像建楼一样建好的地基。
- `xxxx` 就是要创建的项目名。



![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123230913100-1500270612.png)



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123231358449-740612857.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123231547815-1518469652.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123231828958-535144602.png" alt="image" />

当然：上图中的那个babale单词可能是错的，我记不清了（也懒得百度）^ _ ^ 。反正单词大概是那个。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123232242209-1195281782.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123232442821-786002566.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220123232528562-387534918.png" alt="image" />

上面这些记不住没关系，在没选择安路由之前一路回车，然后开始选择时一路no，最后选择use npm即可。后续把东西学完了，那就可以把对应的东西装上了。

拉取完了之后，就是下面的样子：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124000435774-1577274690.png" alt="image" />



然后进到创建的项目中，使用 `npm run dev` ，会得到一个网址，浏览器访问就可以了

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124000634231-347325800.png" alt="image" />



上面这个窗口别关了啊，不然使用地址访问不了的。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124000820652-1925980113.png" alt="image" />









#### 分析cli构建的项目

使用vscode打开刚刚编译的项目。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124131334491-1962512195.png" alt="image" />







1. **package-lock.json**



![image-20240215105805302](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215105735602-1056861838.png)





2. **package.json**



![image-20240215105924058](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215105854142-958398108.png)



> 以上就是基础的东西，接下来就对照前面手写的单文件组件思路来分析接下来的东西，那时说过：`main.js`是入口，所以cli脚手架程序就从`main.js`入手（在cli中为什么它是入口？脚手架底层做的处理）。

1. **main.js**



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124133345173-115160960.png" alt="image" />



2. **引入了App.vue组件，那就接着分析App.vue组件**



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124133732427-1649975600.png" alt="image" />

**注意：**

- assets目录是专门用来放静态资源的，如：png、mp3...（后端的人就把这个目录当做是SpringBoot中的那个static目录）。
- components目录是专门用来放组件的（App.vue是一人之下【vm】万人之上【其他任何组件】，所以不在这里面）。



3. 上面引入了helloword组件，而那里面就是一堆正常的组件写法

4. **分析得差不多了，但是：还少了一个重要的东西，容器在哪里？**

就在`index.html`中，而这个东西有一个专门的public目录来放

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124140454984-928687964.png" alt="image" />

经过前面的分析之后，再结合上次写的单文件组件，就知道对应的东西写在哪里了。

> 顺便说一下：vscode中启动vue程序，按ctrl + `飘字符（ esc下面、tab上面的那个按键）唤出控制台，后面的就知道怎么做了（做了修改之后，按ctril+s保存时会自动重新编译）。@紫邪情





### 认识 render() 函数

把前面编译好的例子改一下， 我的是把前面疏通流程的代码拷贝进来的。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124145334881-365075270.png" alt="image" />



![image-20240215111748959](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215111719297-2038401167.png)



原因就是：代码中的一句代码。

![image-20240215111848650](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215111818651-1855076106.png)

那就去看一下vue到底有哪些版本？按住ctrl+鼠标点引入的vue哪里，点vue就进去了。

![image-20240215111934090](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215111904158-702208361.png)



点开它的包说明：就发现引入的是 dist/vue.runtime.esm.js

![image-20240215112024196](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215111954344-1861399081.png)

随便选择一个右键在资源管理器中显示，就可以看到文件大小（可以和vue.js对比，就少了100kb作用而已，少的就是模板解析器，但是：少的这部分用处很大）

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124155500562-183094492.png" alt="image" />







> vue为什么要搞出这么多版本？

- 这是因为vue其实就是将webpack进行了封装，然后添加了一些技术，从而搞出来的，所以webpack在进行打包时会将`.vue`转成`.js`从而实现渲染（后端人员不懂webpack的，就粗暴地把它当做是maven中的install打包，当然：compile编译等等这些指令功能也在vue中有相同的效果的实现机制）。
- 而程序编写完了之后，webpack本身就支持将`.vue`转成`.js`，那使用webpack打包时，模板解析器就不应该出现了（模板解析器就是编写时解析而已），所以真实打包时如果出现了模板解析器就会出现一个骚气的事情，举个例子：去做某事。



![image](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215112349494-1596396286.png)

- 整出那么多版本的原因就知道了呗，减少程序体积、提高性能嘛。







**回到前面报的错，怎么解决？控制台已经把答案给的很明确了**。

1. 使用包含模板解析器的vue就可以了。
2. 使用`render()`函数实现模板解析器的效果。







1. **使用包含模板解析器的vue最简单**。引入的vue有问题，那就改一下嘛

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124161743678-1801269679.png" alt="image" />



![image-20240215113029456](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215112959421-468320953.png)





2. **使用`render()`函数来实现解析template的功能**



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124162434252-160461777.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124162540580-913715600.png" alt="image" />



知道了`render()`的结构，那就去实现解析template的功能

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124162919511-179701005.png" alt="image" />



然后兰姆达表达式简写不就成原来的样子了吗

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124163347741-264128374.png" alt="image" />



- App就是h2，因为：App是一个组件，使用时就是`<App/>` 或 `<App><App>`。
- 这个`render()`函数也就在vm绑定容器时会用到，其他地方见都见不到的。





### 关闭语法检测

官网中有，改cli的全局配置也是一样的套路。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124191719808-1461663237.png" alt="image" />



> 官网里面就是说新建一个 `vue.config.js` 的文件，这个文件必须和`package.json`是同级目录，然后要配置东西时，找对应的配置项，最后复制粘贴对应的内容（或改动一点自己要的配置项即可）。

关闭语法监测，就是名为lintOnSave的配置项罢了。

创建`vue.config.js`文件：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124192225417-1234711484.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124193229516-1767731080.png" alt="image" />

经过上面的操作之后，就算定义一个变量，然后后面一直没用这个变量也不会导致项目启动不了了，否则：使用`npm run serve`时，就会报错，导致启动不了。

而创建这个`vue.config.js`文件能够修改cli的配置是因为：cli是在webpack的基础上做出来的，而webpack是在nodejs的基础上整出来的，因此：程序最后是将自己写的`vue.config.js`去和webpack中的进行了合并，从而就成功了。

> 注意点：cli中不是什么都可以修改的。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220124194042469-586304610.png" alt="image" />

想要重新改动，就在刚刚新建的`vue.config.js`中配置即可，但是注意：

- `vue.config.js`每一次更新都要使用`npm run serve`重新启动项目，否则不生效。
- `vue.config.js`中不可以说：不用了加个注释，然后等后面用的时候再解开注释，这种是不行的，要么就配置，要么就不配置，否则：启动不了项目。



最后：vue隐藏了webpack的配置，要想查看默认配置的话，那就使用 ctrl+飘字符 唤出控制台后，输入：`vue inspect > output.js`，然后就会在项目中生成一个`output.js`文件，里面就有默认配置。





### VSCode设置Vue文件模板

![image-20241021165924409](https://img2023.cnblogs.com/blog/2421736/202410/2421736-20241021165953159-993686082.png)

![image-20241021170051226](https://img2023.cnblogs.com/blog/2421736/202410/2421736-20241021170050609-131398982.png)



![image-20241021170156982](https://img2023.cnblogs.com/blog/2421736/202410/2421736-20241021170156239-1518228972.png)



文件内容如下：

> `prefix` 为你要使用的快捷键

```json
{
    "vue文件自定义模板": {
        "prefix": "vue",
        "body": [
            "<template>",
            "  <div class='$1'></div>",
            "</template>",
            "",
            "<script>",
            "  /* 这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）",
            "   * 例如：import 组件名称 from '组件路径';",
			"   */",
            "",
            "  export default {",
            "    // import引入的组件需要注入到对象中才能使用",
			"",
            "    components: {},",
            "    data() {",
            "      // 这里存放数据",
            "      return {",
            "      };",
            "    },",
			"",
            "    // 监听属性 类似于data概念",
            "    computed: {},",
            "    // 监控data中的数据变化",
            "    watch: {},",
            "    //方法集合",
            "    methods: {",
            "    },",
			"",
            "    // 生命周期 - 创建完成（可以访问当前this实例）",
            "    created() {",
            "    },",
            "    // 生命周期 - 挂载完成（可以访问DOM元素）",
            "    mounted() {",
            "    },",
			"    // 生命周期 - 创建之前",
            "    beforeCreate() {},",
			"    // 生命周期 - 挂载之前",
            "    beforeMount() {},",
			"    // 生命周期 - 更新之前",
            "    beforeUpdate() {},",
			"    // 生命周期 - 更新之后",
            "    updated() {},",
			"    // 生命周期 - 销毁之前",
            "    beforeDestroy() {},",
			"    // 生命周期 - 销毁完成",
            "    destroyed() {},",
			"    // 如果页面有keep-alive缓存功能，这个函数会触发",
            "    activated() {}, ",
            "  }",
            "</script>",
			"",
            "<style scoped>",
            "  /* @import url(); 引入公共css类 */",
            "</style>"
        ],
        "description": "生成vue模板"
    },
    "http-get请求": {
	"prefix": "httpget",
	"body": [
		"this.\\$http({",
		"url: this.\\$http.adornUrl(''),",
		"method: 'get',",
		"params: this.\\$http.adornParams({})",
		"}).then(({ data }) => {",
		"})"
	],
	"description": "httpGET请求"
    },
    "http-post请求": {
	"prefix": "httppost",
	"body": [
		"this.\\$http({",
		"url: this.\\$http.adornUrl(''),",
		"method: 'post',",
		"data: this.\\$http.adornData(data, false)",
		"}).then(({ data }) => { });" 
	],
	"description": "httpPOST请求"
    }
}
```



### 认识 ref 属性

> **这个属性是用来给“元素”或“子组件“注册引用信息（也就是id属性的替代者）**，这个小东西很重要，关系到后续组件与组件之间的通信。

重新复制一份src文件夹，用的时候把名字改回来就可以了（ 需要哪一个就把哪一个改为src，然后执行`npm run serve`就行了）

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125165515006-1461581243.png)





运行效果如下：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125170229428-212463198.png" alt="image" />





> 现在有一个需求：获取下图中的DOM结构。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125170146624-1513069445.png" alt="image" />

使用传统js来操作的话，就是`document.getElementById`进行获取，但是：Vue中提供得有ref属性来进行操作：**ref属性用来给“元素”或“子组件“注册引用信息（也就是id属性的替代者）**，所以**来见识第一个“元素”注册引用信息（在HTML元素中这个ref属性就和id属性是一样的效果）**

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125174725543-1263064583.png" alt="image" />



**第二种：ref属性是在子组件上的**。这种很重要，后面组件与组件之间交互的基础。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125175255026-1281822861.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125175520825-1375854787.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125175810510-1297581874.png" alt="image" />



**但是：这种和id就不同了，id是直接获得了子组件的DOM结构，而ref是获得了组件本身VueComponent**。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125180337682-1244459103.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125180406421-130158996.png" alt="image" />







> 小结：ref属性

- 被用来给元素（id的替代者）或子组件注册引用信息。
- 应用在HTML标签上获取的是真实DOM元素，应用在组件标签上获取的是组件实例对象（vc）。
- 使用方式：
  - 做标识： `<h1 ref="xxx">......<h1>` 或 `<Person ref="xxx"><Person>。`
  - 获取：`this.$refs.xxx`。





### props 配置 - 获取外传数据

> 组件中的props配置就是为了获取从外部传到组件中的数据。

**这个东西很有用，后续玩子传父、父传子就需要这个东西，而且开发中这个东西会经常看到**。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125205905948-774319091.png" alt="image" />





#### 只接收数据

1. **`Person.vue`组件编写内容**

```html
<template>
  <div>
      <h2>{{name}}</h2>
      <h2>{{sex}}</h2>
      <h2>{{age}}</h2>
      <h2>{{job}}</h2>
      <h2>{{address}}</h2>
  </div>
</template>

<script>
    export default {
        name: 'person',
        
        // 使用props配置，使这个Person组件中的数据从外部传进来（封装的思想来咯）
        // 第一种方式：只接收数据即可（数组写法）	此种方式：接收的数据统统都是字符串
        props: ['name','sex','age','job','address']
    }
</script>
```

2. **`App.vue`组件编写内容**

```html
<template>
  <div>
      <h1 ref="content">欢迎来到对抗路，对手信息如下</h1>
      <!-- 使用组件 并 传入数据 -->
      <Person name="紫邪情" sex="女" age="18" job="java" address="浙江杭州"/>
  </div>
</template>

<script>
    import Person from "./components/Person.vue"

    export default {
        name: 'App',
        components: {Person},
    }
</script>
```

3. **ctrl+s重新编译**



效果如下：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125211331501-394623151.png" alt="image" />







#### 接收数据 + 数据类型限定

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125211750516-2033053660.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125212150003-399623477.png" alt="image" />







> 至于限定类型有哪些？ 可以是下列原生构造函数中的一种 。

```javascript
String

Number

Boolean

Array

Object

Date

Function

Symbol

任何自定义构造函数 或 上述内容组成的数组 
```



#### 接收数据 + 限定类型 + 数据有无接收的必要 + 数据默认值

这个东西，玩Java的人看起来很熟悉，和Elastic-Search中的mapping映射很像

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125213211929-312932137.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125213345579-104959398.png" alt="image" />





#### 处理外部传入数据类型问题

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125213836752-388137275.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125213900039-515003699.png" alt="image" />





#### 解决props接收数据之后，修改它的值

props配置中不是什么属性名的值都可以接收的，如：key、ref。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125215039684-1832642613.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125215222367-2013917148.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125215248952-1662209313.png" alt="image" />



意思就是：key不能作为props接收的数据，原因就是因为：key这个属性被Vue给征用了，Vue底层需要使用diff算法嘛，它用了这个key关键字，所以我们不可以用它。





> 回到正题，props接收了数据怎么修改它？
>
> - 首先要知道，props被底层监视了的，所以它的东西只可以接收，不可以修改，想要接收了数据，再修改它，我们就需要借助data，中转一下。

1. 先来看不中转，然后修改props中数据的下场。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125221526306-1840441672.png" alt="image" />



2. 使用data中转，从而修改数据。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125222448248-372621466.png" alt="image" />



#### props配置总结

- 功能：让组件接收外部传进来的数据。

- 传递数据：`<Person name="xxxx">`。

- 接收数据

  - 1）、只接收：`props: ['name']`。

  - 2）、接收数据 + 数据类型限定




  ```javascript
  props: {
      name:String
  }
  ```

  - 3)、接收数据 + 数据类型限定 + 必要性限制 + 数据默认值




```javascript
props: {
    type:String,
    required:true,
    default:'紫邪情'		注：一个数据字段不会同时出现required和defautle
}
```

- 注意：props中的数据是只读的，Vue底层会监测对props的修改，如果进行了修改，就会发出警告。
  - 如果业务需要修改，则：把props中的数据复制一份到data中，然后去修改data中的数据就可以了。
  - 须知：vue中优先使用props中的数据，然后再使用data中的数据（ 可以让data中的数据和props中的数据一样，然后去渲染，发现渲染出来的数据是props中的）。





### mixin 混入 / 混合

这个东西就是把多个组件中共同的配置给抽取出来，然后单独弄成一个js文件，使用一个对象把相同配置给封装起来，然后在需要的组件中引入、使用mixins配置进行使用即可。

这种思想再熟悉不过了，工具类的编写不就是这么回事吗。当然这样的需求一般都用后续要玩的“状态机Vuex”了。



#### 使用

基础代码

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125234214737-1481998040.png" alt="image" />

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125234256448-796833478.png" alt="image" />



在上面的代码中，methods中的代码是相同的，因此：使用mixin混入来进行简化，也是三板斧而已。

1. **新建js文件（名字根据需要取即可）**。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125234948226-761121737.png" alt="image" />



2. **在需要的组件中引入抽取的代码 和 利用mixins配置进行使用**。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125235844650-1906525931.png" alt="image" />



3. 运行效果



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220125235922284-570346451.png" alt="image" />





> mixin混入的一些注意点。

1. 除了生命周期，如果其他配置是在mixin中定义了，同时在组件中也定义了，那么：优先使用的是组件中自己定义的（无论二者相同与否都一样）。

2. 如果在mixin中定义了生命周期钩子函数，那么：优先使用mixin中的钩子函数。


如果二者不同，那么就会造成二者中定义的都会被调用。





> mixin混入是支持全局配置的，不过这种操作不当会出现问题。

因此：目前不建议用，思路如下：

1. 一样的创建js文件。
2. 在`App.vue`中引入。
3. 在`App.vue`中使用`vue.mixin( 暴露对象1 )` 、`vue.mixin( 暴露对象2 )`.......。

- 使用了这三板斧之后，就可以实现全局配置了。





#### mixin 混入总结

功能：把多个组件共同的配置提取成一个混入对象

使用方法：

1.  新建`xxx.js` 定义混入，如：



```javascript
暴露方式 const 对象名 {
    data(){......},
     methods:{........},
     ........
}
```

2. 在需要的组件中引入混入。

3. 在需要使用混入中定义的功能的组件中使用混入，如：

- 1）、局部混入：`mixins: [ xxxxxx , ......... ] `  ， 注意：这里必须用数组。
- 2）、全局混入：`Vue.mixin( xxxxx )`。





### 插件

这个东西的作用很大，它可以合理的增强Vue。



**使用，还是三板斧**

1. 创建js文件（包含`install()`方法的一个对象）。

2. 引入插件。
3. 使用插件。



#### 玩一下插件

基础代码效果

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126165956728-1132781743.png" alt="image" />



1. **创建一个包含` install()` 方法的对象的js文件**。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126170031243-574448295.png" alt="image" />



2. **在`main.js`中引入、使用插件**。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126170251288-2089624736.png" alt="image" />



3. 效果如下



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126170406161-883176753.png" alt="image" />



> 可以得知：创建的插件中传递的那个参数Vue就是vm（Vue实例对象）的缔造者 —— vue构造函数。

得到上面那个结果就很重要了，试想：我们使用这个构造函数做过什么事？
- 自定义全局指令 `Vue.directive`。
- 定义全局过滤器 `Vue.filter`。
- 定义全局混入 `Vue.mixin`。
- ........

- 把这些东西放到创建插件的`install()`中可行？它接受的参数就是Vue嘛。







> 正宗玩法

1. 创建包含`install()`方法的对象的js文件。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126190907159-439134576.png" alt="image" />



2. 在`main.js`中引入插件、应用组件。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126191141701-220958625.png" alt="image" />



3. 使用插件中的东西。



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126191252611-967272338.png" alt="image" />



4. 效果如下：



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126191346550-1361854856.png" alt="image" />



当然：我们也可以给插件中传东西进去。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126191906110-73038836.png" alt="image" s/>



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126192143010-1266926236.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126192230618-824451287.png" alt="image" />





#### 插件总结

功能：用于增强Vue。

本质：是包含`install()`方法的一个对象的js文件，install的第一个参数是Vue，第二个以后的参数是插件使用者传递的数据。

插件的玩法：

1. 定义插件：



```javascript
// 1、创建插件  export default 是自己选择的暴露方式
export default {
    install(Vue,[ other params ]){
        
        // 定义全局过滤器
        Vue.filter( ..... ),
            
        Vue.directive( ...... ),
                   
        Vue.mixin( ....... )
        .........
    }
}
```

2. 在`main.js`中引入插件。

3. 在`main.js`中向Vue应用插件  `Vue.use( 插件名 )`。

4. [ 使用插件中定义的东西 ] ———— 可有可无，看自己的代码情况。

- 这里注意一个东西：定义插件中的`install()`第一个参数是Vue，即：vm的缔造者，Vue构造函数（这里可以联想原型对象，也就是前面说的vm和vc的内置关系：`VueComponent.prototype._ _proto _ _ === Vue.prototype`），这也就是说在Vue身上加点东西，那么：vm和vc都可以拿到，如：




```javascript
Vue.prototype.$myMethod = function(){ ...... }
Vue.prototype.$myProperty = xxxx


prototype		路线是加东西

_ _proto_ _		路线是取东西
```





### scoped 局部样式

作用：让样式在局部生效，防止冲突。

写法：`<style scoped>。`



假如有两个组件，里面都用了同一个class类名，但是做的style却是不一样的。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126205305947-257242726.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126205423582-767394005.png" alt="image" />





此时如果把两个class名字改成一样呢？开发中样式多了这种事情是在所难免的。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126205549405-1022484063.png" alt="image" />





凭什么就是`Person2.vue`组件中的样式优先？这和App.vue中引入组件的顺序有关。

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126205807434-728000752.png)



不然：把组件引入的顺序换一下就发现变了。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126205900959-449829121.png" alt="image" />





那上述样式冲突了怎么解决？答案就是使用scoped限制作用域（后端人员，这个东西就类似maven中依赖的作用域，是一样的效果）

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126210306871-1906773058.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126210342684-1545689669.png" alt="image" />





当然：style标签不止支持scoped属性，还可以用其他的。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126210911888-1716479900.png" alt="image" />



> 另外：**less需要less-loader支持**，所以需要安装less-loader，但是：有坑（版本的问题）。

```bash
# 控制台输入
npm view webpack versions		# 查看webpack所有版本（旧+新）
```

![image-20240215124834496](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215124805109-1078804287.png)



cli中的webpack版本：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220126212122785-901884173.png" alt="image" />



安装适合cli的webpack的less-loader版本：

```bash
npm install less-loader@版本号
```

![image-20240215125052584](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215125022734-2040476453.png)







> 最后还有一个问题：scoped使用在`App.vue`中就会发生很诡异的事情。

- `App.vue`是大哥，所以这里面的style会全局生效，也就是多组件都在使用的样式，就可以提到`App.vue`中写，然后在其他需要的组件中使用即可 。
- 但是：如果`App.vue`中style使用了scoped，那么就会导致：样式只在`App.vue`中有效，那么：其他组件中想要用，对不起，管不了，最后页面就会出现诡异的事情 —— 不是自己写的样式的样子。
- 所以：**`App.vue`中最好别用scoped属性，而其他组件中最好都加上scoped属性**。







### 组件化应用应该怎么去写？

实例：实现如下的效果。 就是一个人名录入，然后可以对名字做一点操作罢了。

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127151054273-1318980545.png)

组件编写流程，基本上都是一样的套路，接下来按照套路去弄就可以了。





#### 分析结构 / 拆分组件

根据页面结构来看，可以拆分成如下的结构：

1. 外面大的框就是一个组件 —— 也就是`App.vue`父组件，而App组件中又包含的如下组件：

- 1.1、输入框是一个组件（子）。
- 1.2、人名的展示框是一个组件（子 ，却是下面两个的“父”）
  - 1.2.1、然后发现展示框里面又有每一个人名组件（子）。
  - 1.2.2、还有一个全选 / 显示已选人数的组件（ 子）。



#### 创建对应组件并编写内容

**创建并编写组件对应内容( 统称编写静态组件 ）**

1. **App组件**

- 1）、HTML结构。



```html
<template>
  <div id="root">
      <div class="name-container">
        <div class="name-wrap">

          <NameHeader></NameHeader>
          <NameList></NameList>
          <NameFooter></NameFooter>

        </div>
      </div>
  </div>
</template>

<script>
import NameHeader from "./components/NameHeader.vue"
import NameList from "./components/NameList.vue"
import NameFooter from "./components/NameFooter.vue"

    export default {
        name: 'App',
        components: {NameHeader,NameList,NameFooter}
    }
</script>
```



- 2）、CSS样式 + 后续需要的通用样式。



```css
body{  
  background: #fff;
} 

.btn {
  display: inline-block;
  padding: 4px 12px;
  margin-bottom: 0;
  font-size: 14px;
  line-height: 20px;
  text-align: center;
  vertical-align: middle;
  cursor: pointer;
  box-shadow: inset 0 1px rgba(255,255,255,0.2),0 1px 2px rgba(0, 0, 0, 0.05);
  border-radius: 4px;
}

.btn-danger {
  color: #fff;
  background-color: #da4f49;
  border: 1px solid #bd362f;
}

.btn-danger:hover {
  color: #fff;
  background-color: #bd362f;
}

.btn:focus {
  outline: none;
}

.name-container {
  width: 600px;
  margin: 0 auto;
}

.name-container .todo-wrap {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
}
```



2. 输入框组件。

- 1）、HTML结构。



```html
<div class="name-footer">
    <label>
        <input type="checkbox">
    </label>
    <span>
        <span>已选择0</span> / 共计2
    </span>
    <button class="btn btn-danger">清除已选人员</button>
</div>
```



3. 人名展示框。

- 1）、HTML结构。



```html
<template>
    <NameObj></NameObj>
</template>

<script>
import NameObj from "./NameObj.vue"

    export default {
        name: 'NameList',
        components: {NameObj}
    }
</script>
```



- 2）、CSS样式。



```css
/* #region list */
  .name-main {
      margin-left: 0px;
      border: 1px solid s#ddd;
      border-radius: 2px;
      padding: 0px;
  }

  .name-empty {
      height: 40px;
      line-height: 40px;
      border: 1px solid #ddd;
      border-radius: 2px;
      padding-left: 5px;
      margin-top: 10px;
  }

/* #endregion */
```



4. 每个人名展示。

- 1）、HTML结构



```html
<template>
    <ul class="name-main">
        <li>
            <label>
                <input type="checkbox"/>
                <span>xxxxx</span>
            </label>
            <button class="btn btn-danger" style="display:none">删除</button>
        </li>
    </ul>
</template>

<script>
    export default {
        name: 'NameObj',
    }
</script>
```



- 2）、CSS样式



```css
/* #region item */
  li {
    list-style: none;
    height: 36px;
    line-height: 36px;
    padding: 0 5px;
    border-bottom: 1px solid #ddd;
  }

  li label {
    float: left;
    cursor: pointer;
  }

  li label li input {
    vertical-align: middle;
    margin-right: 6px;
    position: relative;
    top: 1px;
  }

  li button {
    float: right;
    display: none;
    margin-top: 3px;
  }

  li:before {
    content: initial;
  }

  li:last-child {
    border-bottom: none;
  }

/* #endregion */
```





运行效果如下：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127205816624-189722342.png" alt="image" />



自此：静态组件编程就搞定，后续就可以做数据的动态绑定和交互这些了。

**将纯HTML + CSS + JS转成组件化开发也是一样的套路，流程如下：**

1. 把整个HTML结构放到`App.vue`组件的template模板中。

2. 把所有的CSS放到`App.vue`组件的style中。

3. 有JS的话，那么把js文件创好，然后引入到`App.vue`组件中。

4. 开始去分析结构，然后拆分成组件，之后把对应的内容放到对应组件中，最后做后续的动态数据绑定、交互即可。



#### 动态数据绑定

按照分析，要展示的数据是一堆数据，而数据显示的地方是NameList，所以：data数据就放到NameList中去。**本实例中数据放到这里有坑啊，但是：先这么放，后续遇到坑了再调整**，所以改一下源码。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127195833422-1340012636.png" alt="image" />

**数据是弄好了，但是：真正展示数据的是NameObj组件来显示出来的，而NameObj是NamelList的子组件（这就是：父传子），这种就需要借助props配置项（ 从外向内传嘛）**，所以：开始操作。



1. **父传子**



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127200738531-1552759242.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127200854053-368920431.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127205941956-1688018460.png" alt="image" />





2. **获取输入的内容并添加到显示的顶部 —— 和后续知识挂钩的重点来了**。

输入框组件是App组件的子组件，而数据展示是NameList组件中的NameObj组件，即：现在关系就是如下的样子。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127202448353-1636141295.png" alt="image" />

上面这种就是不同组件之间的通信情况，现在来了解原生的解决办法





3. **原生的不同组件通讯**

- `App.vue`是大哥，用它做文章，这样就变成了App这个父组件和输入框以及Namelist这两个子组件之间的关系了。

第一步：将data搬到App组件中去并传给NameList数据展示组件。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127211729839-657203027.png" alt="image" />



然后NameList数据显示区再传给数据显示组件

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127211910732-823764014.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127212445324-2065618402.png" alt="image" />



这样父传子、子传孙....这样就串通一条路了，即：下图右边部分。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127212257578-1329439964.png" alt="image" />



第二步：将输入框组件中的数据收集起来，并传给父组件（这就是子传父）。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127204228819-347115837.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127210128464-581619569.png" alt="image" />







**将数据传给父组件 —— 开始子传父**

> 子传父的技巧就是：父组件传给子组件一个函数（ props的变样版，传的不是对象、key-value，而是整一个函数传过去）、然后子组件利用props接收到函数之后调用一下（把传递数据当做参数），那么父组件中的函数的参数就是要传递的数据了。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127214017548-1274186829.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127221625004-1248940030.png" alt="image" />



上图id是使用`random()`生成的随机数，这有弊端的，数字有穷尽的时候，所以：严格来说用uuid、身份证号、电话号码......作为id最好。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127214850492-2014684179.png" alt="image" />









4. **将子组件传递的数据添加到数据栏的顶部去**

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127215255999-1103295638.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127215325829-807978443.png" alt="image" />













#### 交互编写

##### 实现选择和数据的改变

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127234355353-1787062224.png" alt="image" />







**1）、最简单粗暴的方式 —— 但是：不建议用**

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127234720067-1402567927.png" alt="image"/>



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127234804237-538216818.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220127234851041-160331759.png" alt="image" />



**成功是成功了，但是：说了不建议用，因为：这种方式违背了Vue的设计（但是：开发中又喜欢用，简单实用嘛）**。

- 违背了Vue的设计是因为：props在底层是被检测了的，Vue不支持去修改它的东西，但是：按上述的方式做了之后，却会发现：并不会报错，这是因为：修改的是值和对象的区别。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128000020796-1073366447.png" alt="image" />

<bt />



**2）、按照逻辑老实编写**

这种实现方式的思路就是：在页面中点击 / 改变选择框时，拿到这条数据的id，然后利用id去所有的数据中遍历，找到对应的这条数据，最后改变这条数据isCheck的值即可。

**下面的操作也是一个父传子的使用过程：这里有一句话：数据在哪里，对数据的操作（methods）就在哪里**。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128003321685-39176417.png" alt="image" s/>



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128003445925-1471538691.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128003658152-239266697.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128003848093-1766884282.png" alt="image" />









##### 实现每条数据的删除功能

**1）、先把样式解开，让鼠标悬浮时删除按钮可见。**

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128115000710-561305152.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128115039676-871870857.png" alt="image" />



**2）、实现数据与删除按钮交互（还是子传父的套路）**。

实现逻辑简单：拿到要删除数据的id，然后去众多数据中把要删除数据给过滤掉不显示即可（逻辑删除）。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128120904104-1545725197.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128121017928-188064026.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128121052613-1183138405.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128121140987-1276587346.png" alt="image" />











##### 实现底部的已选择人数 和 总计人数功能

###### 最原生的方式

> 父传子 —— 传递persons这个数据。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128150407096-1185064235.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128150535976-538387020.png" alt="image" />





###### 使用数组的 reduce() API

使用数组的`reduce()`这个API，这个API专做数据统计的。



**认识一下`reduce()`**

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128151708693-844375256.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128151908860-1003725536.png" alt="image" />

**`reduce()`最终的返回值是：程序执行的最后一次的nextValue值**。







**使用`reduce()`实现功能**。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128152729041-1930724157.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128152822215-1661230446.png" alt="image" />



接下来就只剩下底部的全选和清除已选这两个功能了。







##### 实现全选交互 和 清除已选人员功能

###### 实现全选交互

实现全选交互（子传父 + 计算属性使用技巧）：

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128182346665-1668663782.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128182503969-725756059.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128182700635-754395647.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128182805469-1429706630.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128182846190-616125125.png" alt="image" />



上面这个全选使用分步利用子传父实现每一步也是可以的，只是有点复杂罢了。





###### 清除已选人员

这个的思路更简单了，就是查看页面中的数据哪些的isCheck为true，然后过滤掉这些数据即可（实现方式一样简单，也是子传父）。

<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128192746278-1272087066.png" alt="image" />



<img src="https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220128192841734-1525904826.png" alt="image" />





#### 功能改善

**localstorage和sessionstorage**。 前者浏览器关闭不会清空，后者关闭浏览器会清空，二者常用的API都一模一样，**二者里面存的数据都是key-value的形式**。

- 存数据  `setItem（ 'key', 'value' ）` 使用：`LocalStorage.setItem( 'key' , 'value' )` 其他的都是差不多的。
- 取数据 `getItem( 'key' )`

注：若key不存在，则：此API返回值是null；若key是对象字符串  那么用`JSON.parse()`转成对象，返回值也是null。

- 删除某一个数据 `removeItem()`。

- 清空所有 `clear()`。

二者存储的内容大小一般为5M的字符串（ 不同浏览器可能会不一样 ）。

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220129144032680-1547509565.png)

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220129144157987-1764888291.png)







#### 组件化开发流程总结

**组件化编写流程：**

1. 拆分静态组件：组件要按照功能点拆分，命名不要和HTML元素名冲突。

2. 实现动态组件：考虑好数据的存放位置，数据是一个组件在用，还是一些组件在用。

- 一个组件在用：把数据放到组件自身即可。
- 一些组件在用：把数据放到它们共同的父组件上【 这也叫状态提升 】。

3. 实现交互：从绑定事件开始。





**props配置总结：适用于以下过程**

1. 父组件 ——> 子组件  通信。

2. 子组件 ——> 父组件 通信。



**v-model总结**

- 使用v-model时要切记：v-model绑定的是值，但是：这个值不能是props中传递过来的值，因为：props底层被Vue监测了的，不允许修改。
  

> 注：props中传过来的若是对象类型的值时，虽然修改"对象中的属性"时Vue不会报错，但是：不建议用。







## 高级篇

### 组件的自定义事件

#### 绑定自定义事件

> 这玩意儿也是为了子父组件间的通信。

这里实现方式有两种：

- 第一种：用 `v-on` + `VueComponent.$emit`实现【PS：此种方式有点类似子传父 】；
- 第二种：用 `ref`属性 + `mounted()`来实现【 此种方式：复杂一点，但是更灵活 】。

两种实现方式都可以，而二者的区别和computed与watch的区别很像。

- PS：第二种方式可以实现异步操作，如：等到ajax发送请求得到数据之后，再进行事件绑定。



接下来看实例，从而了解更明确点，用如下实例做演示：

- PS：不用前面玩的子传父，但是：自行可以先回顾一下子传父的传统方式。

![image](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215140227103-659984014.png)



##### v-on + VueComponent.$emit 实现

> 父给子一个函数，子执行，参数即为父要得到的数据。



![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220130201643486-1150468967.png)



![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220130201755940-1772527908.png)





![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220130202510598-1864617529.png)





##### ref属性 + mounted() 实现

在前面实现的基础上，子组件代码不变，在父组件中加入如下的代码：

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220130204028893-701628692.png)



![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220130204138201-971219193.png)

`mounted()`中是可以进行异步操作的，所以才可以让这种自定义事件更灵活。



**另外：既然是事件，那么也可以使用事件修饰符：prevent、stop、once**

- 在 `v-on` 中，这三者和以前一样的玩法，都是加在事件名后面即可，如：`@zixeiqing.once = "xxxxx"`。
- 在ref属性中，是用在`this.$refs.person.$on('zixieqing',this.demo )`中的`$on`这里的，once就是使用`$once`，替换掉原来的`$on`。





#### 解绑自定义事件

> 这玩意用的就是 `VueComponent.$off( ['要解绑的事件名'] )` 这个内置函数来实现解绑的。
>
> 
>
> 自定义事件的核心话：给谁绑定事件，那么事件就在谁身上；给谁解绑自定义事件，那么就去谁身上解绑。

- 当然：数组`[ ]`中，如果是解绑单个事件，那么`[ ]`这个括号不要也行；
- 如果是解绑多个自定义事件，那么使用 `,` 逗号隔开即可；
- 另外：`$off()`不传递参数时，默认是把组件的所有自定义事件都解绑了（PS：有一个解绑一个）；



![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220130225048994-696640538.png)







> 另外：前面玩Vue生命周期的beforeDestroy时有一个小点只是简单提了一下，生命周期图如下：

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220119133045755-824545654.png)

上图中的内容，有最后一点没有验证：

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220130225740274-1600607693.png)



说在beforeDestroy中，会销毁子组件和自定义事件、说此时销毁自定义事件：

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220130231934069-1974830615.png)



![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220130231840043-788802908.png)



而所谓的销毁子组件也就好理解了，就是把父组件销毁之后，那么：子组件也活不成了。

- PS：要验证的话，可以使用销毁vm，然后看旗下的子组件还能活不？答案肯定是活不成的。







#### 自定义事件中的两大坑

子组件是如下的样子：

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131130211441-1294857331.png)







##### ref属性中，this指向问题

> 在ref属性实现的方式中，关于this的指向问题。

1. 第一种就是将回调函数放到父组件的methods中。



![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131125832987-1190794306.png)

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131125948270-39290130.png)



> 此种方式，会发现this指向的是父组件。







2. 第二种：将回调函数直接放到 `this.$refs.people.$on( 'event',xxxx ) ]` 中的xxxx中。



![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131130917716-1361511934.png)

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131131111874-1383805561.png)





> 这种情况：会发现，this不再是父组件实例对象，而是子组件的实例对象，但是：可以让它变为子组件实例对象。
>
> - PS：把回调的普通函数写成兰姆达表达式就可以了。

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131131359393-1243885476.png)

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131131529444-1034994717.png)









##### 组件用原生DOM事件（native修饰符）

> 组件使用原生DOM事件的坑【PS：了解native修饰符 】。

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131140540121-1948122370.png)

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131140640428-1218124486.png)

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131140756095-1782748247.png)

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131140947144-1114589646.png)





#### 自定义事件总结

1. 自定义事件是一种组件间通信的方式，适用于：子组件 ——> 父组件通信。
2. 使用场景：想让子组件给父组件传递数据时，就在父组件中给子组件绑定自定义事件（PS：事件回调在父组件methods / 其他地方 中），而要解绑自定义事件就找子组件本身。

3. 绑定自定义事件：

- 1）、在父组件中：`<Person @zixieqing="demo"/>` 或 `<Person v-on:zixieqing="demo"/>`。

- 2）、在父组件中：



```javascript
<Person ref = "demo"/>
	.........
methods: {
	test(){......}
}
	.........
mounted(){
	this.$refs.demo.$on('eventName',this.test)
}
```

- 3)、若想让自定义事件只能触发一次，可以使用`once`修饰符【PS：使用v-on的方式实现的那种 】 或 `$once`【PS：使用ref属性实现方式的那种 】。



4. 触发自定义事件： `this.$emit('eventName',sendData)`  【PS：给谁绑定自定义事件，就找谁去触发 】。
5. 解绑自定义事件：`this.$off(['eventName',.......])`   【PS：给谁绑定自定义事件，就找谁解绑；另：注意解绑事件是单个、多个、全解的写法 】。
6. 组件上也可以绑定元素DOM事件，但是：需要使用native修饰符。
7. 注意项：通过`this.$refs.xxxx.$on('eventName',回调)`绑定自定义事件时，回调要么配置在父组件的methods中，要么用兰姆达表达式【PS：或箭头函数 】，否则：this执行会出现问题。









### 全局事件总线

这玩意儿吧，不算知识点，是开发中使用的技巧而已，里面包含的知识在前面全都玩过了，只是：把知识做了巧妙的应用，**把自定义事件变化一下，然后加上Vue中的一个内置关系`VueComponent.prototype._ _proto _ _ === Vue.prototype`从而实现出来的一个开发技巧**。

> 此技巧：可以实现任意组件间的通信。





#### 疏通全局事件总线逻辑

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131171741383-1848085067.png)



但是：现在把思路换一下来实现它

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131192618442-315592341.png)







> 通过上面的分析图了解之后，就可以分析出，单独选取的那个组件需要具有如下的特性：

1. 此组件能够被所有的组件看到。
2. 此组件可以调用`$on()`、`$emit()`、`$off()`。





##### 能让所有组件都看得到

那么为了实现上一节结论的第一步：能够让所有的组件都看得到，可以怎么做？

1. 使用window对象，可以做到，但是：不推荐用。



![image](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215143305494-93458610.png)

![image](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215143319576-2015927089.png)

![image](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215143336368-1940403285.png)





> 此种方式不推荐用呢，是因为：本来就是框架，谁还去window上放点东西呀，不是找事吗。







2. 就是利用Vue中的内置关系`VueComponent.prototype._ _proto _ _ === Vue.prototype`。

即：公共组件选为Vue实例对象vm。这个关系怎么来的，这里不再说明了，在基础篇`VueComponent()`中已经说明过了，**利用此内置关系就是利用：VueComponent可以获取Vue原型上的属性和方法，同时选取了Vue实例对象之后，`$on()`、`$emit()`、`$off()`都可以调用了，这些本来就是Vue的内置函数，Vue实例对象还没有这些函数吗**。











#### 全局事件总线实例

> 实现方式：
>
> - vm + `beforeCreate()`【ps：初始化嘛，让关系在一开始就建立】；
>
> - `mounted()` + ` $on()` + `$emit()` + `beforeDestroy()`【ps：做收尾工作，解绑自定义事件 】+ `$off()`。



实例演示：

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131195544142-964069282.png)

> 注：上图中的`$bus`是自己起的名字，开发中一般起的都是这个名字，bus公交车嘛，谁都可以上，而且还可以载很多人，放到组件中就是：谁都可以访问嘛，加了一个`$`是因为迎合Vue的设计，内置函数嘛，假装不是程序员自己设计的（PS：实际上，bus还有总线的意思）。

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131202831416-1091447486.png)

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131202933284-132863941.png)

![image](https://img2022.cnblogs.com/blog/2421736/202201/2421736-20220131203333762-1848739113.png)









#### 全局事件总线总结

1. 全局事件总线又名GlobalEventBus。

1. 它是一种组件间的通信方式，可以适用于任何组件间通信。

3. 全局事件总线的玩法：

- 1)、安装全局事件总线。



```javascript
new Vue({
    .......
    beforeCreate(){
        Vue.prototype.$bus = this
    },
    ......
})
```



- 2）、使用事件总线。
  - 发送数据：`this.$bus.$emit('EventName',sendData)`

  - 接收数据：A组件想接收数据，则：在A组件中给$bus绑定自定义事件，把事件的回调放到A组件自身中【PS：靠回调来得到数据 】




```javascript
// 使用methods也行；		不使用，把回调放到$on()中也可以		推荐使用methods，因为不必考虑$on()中的this问题
methods: {
    sendData(){ ...... }
},
........
mounted(){
    this.$bus.$on('eventName',receiveData)
},
.......
beforeDestroy(){
    this.$bus.$off([ 'eventName' , ..... ])
}
```







### 消息订阅与发布

什么是消息订阅与发布？

- 这个东西每天都见到，就是：关注，关注了人，那别人发了一个通知 / 文章，自己就可以收到，这就是订阅与发布嘛。
- **这里使用pubsub-js这个库来演示（使用其他库也行，这些玩意儿的思路都一样 ，这是第三方库啊，不是vue自己的）**，其中：
  - pub：就是publish，推送、发布的意思。
  - sub：就是subscribe，订阅的意思。
  - 也就是：一方发布、一方订阅嘛。







#### 玩一下pubsub-js

基础代码：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220201174137363-46336343.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220201174210974-2018307169.png)







1. **给项目安装pubsub-js库**。指令：`npm install pubsub-js`



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220201174345898-1164344392.png)





2. **消息发布方**

- 2.1、引入pubsub-js库。
- 2.2、使用 `publish( 'msgName' , sendData )` 这个API进行数据发送。



![image](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215144919527-518718000.png)





3. **消息接收方**

- 3.1、引入pubsub-js库。
- 3.2、使用 `subscribe( 'msgName' , callback )` 这个API利用回调进行数据接收。
- 3.3、关闭订阅。



![image](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215145019457-688369348.png)





效果如下：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220201180122780-1440188599.png)











#### 消息订阅与发布总结

- 它是一种组件间通信的方式，适用于：任意组件间通信。

- 使用步骤：

  - 1）、安装pubsub-js 指令：`npm install pubsub-js`。

  - 2）、消息接收方、发送方都要引入pubsub  代码;：`import pubsub from "pubsub-js"`。
    - 数据发送方：`pubsub.publish('msgName',sendData)`。
    
    - 数据接收方：
    



```javascript

// methods可写可不写		推荐写，不用考虑this的指向问题，和自定义事件一样的
methods: {
	demo(){.....}
}
........
mounted(){
	// 使用this.msgName把每条订阅都绑在组件实例对象vc上，方便取消订阅时获取到这个订阅id
	this.msgName = pubsub.subscribe('msgName', callback)	// 如果不写methods，那么回调就写在这里，注意：使用箭头函数
}
.......
beforeDestroy(){
	pubsub.unsubscribe( this.msgName )
}
```







### 认识$nextTick()

- 用法：`this.$nextTick(回调函数)`。
- 作用：在下一次DOM更新结束后执行其指定的回调。

PS：让代码的解析产生时间差，因为：有些东西等一部分视图已经出来了再把弄外的东西加上，如：input输入框，聚焦的问题，就需要等input到了页面，然后再把焦点放上去。

- 什么时候用：在改变数据后，要基于更新后的新DOM进行某些操作，要在nextTick所指定的回调函数中执行（setTimeout定时器也可以做，只是官网推荐用这个）。







### Vue封装的过度与动画

#### Vue中实现动画的套路

样式还是要自己写，只是：调用样式的逻辑Vue进行了封装，只需要按照标准把东西配好，Vue就会自动去帮忙执行动画的样式。

```html
<template>
    <div>
        <button @click="isShow = ! isShow">显示/隐藏</button>
        <!-- 
            使用一个Vue的特定标签【PS：只限于标签中只有一个元素要有动画时（ 即：h2中的东西 ） 】
            【PS：和template标签一样，在页面中不会解析，是Vue中特定的名字而已 】
            另外：还有一个标签transition-group是玩多个元素都有过度效果的（ 如：多个h2元素也要过度 ）
            【PS：但是需要给每个元素匹配一个key="xxx"属性，此种方式：可以实现多个元素相反过度，一个元素怎样
            另一个元素就不怎样之类的 】
         -->
        <transition appear>
            <!-- 
                谁要实现动画，那就把这个标签套在谁身上，其中：appear是设置 页面初始状态的，
                    即：页面一开始就有动静效果【 设置等价于 :appear = "true"  】
                另外：此标签还有一个name属性  
                但是：这里如果使用了name="xxx"，
                那么：下面Vue套路处的v-enter-active中的v就要换成xxx，不写name默认就是v
                下面都是自己写动画：可以用第三方库，如：npm中的animate.css进去就知道怎么玩了
             -->
            <h2 v-show="isShow">玩Vue中编写动画的套路</h2>
        </transition>
    </div>
</template>

<script>
    export default {
        name: 'Cartoon',
        data() {
            return {
                isShow: true
            }
        },
    }
</script>

<style scoped>
    /* 1、动画样式还是要自己写 */
    h2 {
        background: purple;
        color: white;
    }
    @keyframes zixieqing {
        /* 从哪里来？ */
        from {
            transform: translateX( -100px )
        }
        /* 到哪里去？ */
        to {
            transform:translateX( 0px )
        }
    }


    /* 
      2、使用Vue的套路，让其执行动画的逻辑 
        下面的两个名字是固定的，不能改
            【ps： 若transition中加了name="xxx"，那么下面的v必须为xxx，否则：不生效】
            实现动画用下面两个类名即可，要实现过度效果，需要再借助另外的类名：v-enter、v-enter-to 和 v-leave、v-leave-to
    */
    .v-enter-active {
        animation: zixieqing 0.5s linear;
    }
    .v-leave-active {
        animation: zixieqing 1s reverse;
    }
</style>
```







#### Vue中的过渡效果

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202124510765-1969110785.png)





#### Vue封装的过度 与 动画总结

作用：在插入、更新或移除DOM元素时，在合适的时候给元素添加"样式类名"。

官网中的图示：



![image](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215145643774-657641733.png)



写法：

1. 自己准备好样式

- 元素进入的样式（下面的类名不可以改，把对应样式写在里面，然后搭配 transition / transition-group 标签即可）：
  - 1）、v-enter：进入的起点。
  - 2）、v-enter-active：进入过程中。
  - 3）、v-enter-to：进入的终点。
- 元素离开的样式：
  - 1）、v-leave：元素离开的起点。
  - 2）、v-leave-active：元素离开过程中。
  - 3）、v-leave-to：元素离开的终点。



2. 使用 transition 标签包裹要过度的元素，并配置 name 属性：



```html
<transition name="demo">
	<h2 v-show="isShow">
        这是单个元素过度使用的transition标签
    </h2>
</transition>
```

- 备注：若有多个元素需要进行过度（如上面例子中还要多个h2元素），则：需要使用 `<transition-group>`来包裹过度元素，且每个元素都要使用`key="xxx"`属性来指定唯一标识。



另外：可以使用第三方库；如：npm中的`animate.css`。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202132207575-330576431.png)



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202132330714-1230879286.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202132659223-381101756.png)







### Vue中解决跨域问题

学了Nginx，那么熟悉得不得了，为什么有Nginx，一是跨域、二是解决负载均衡的问题。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202150901689-487664917.png)



#### 使用vue cli 的devServer.proxy配置

这个东西在vue cli官网中有（**ps：是修改默认配置`vue.config.js`中的一种配置**）。



1. 配置默认配置



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202145134442-376391.png)







2. 使用axios发起请求

- 给项目安装axios插件。指令：`npm install axios`

- 在代码中引入axios。代码：`import axios from "axios"`

- 使用axios。代码：




```javascript
axios.get / post('ip:port/sourceName')，then（    // port是请求地的port，如：8080 ——> 8081，此时port就是8080
    // 成功时
    response =>{
        // response是一个对象，数据在这个对象的data中
        console.log( response.data )
    },

    // 失败时
    error => {
        // error也是一个对象，失败信息在message中
        console.log( error.message )
    }
）
```



3. 经过上述操作就可解决跨域问题，但是有缺陷：

**1）、使用vue cli解决跨域时，只能请求一台服务器**。



![image](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215150510843-1371891339.png)

**2）、如果发起请求的地方中已经有了要请求的sourceName资源名，那么：优先获取发起请求地中的资源**。



![image](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215150631918-1455841877.png)



那么就会导致：vue不会去vue cli中开启的代理服务器获取资源，而是直接在本地获取









#### 使用进化版 devServer.proxy

1. 配置默认设置`vue.config.js`



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202153129247-972964804.png)



```javascript
module.exports = {
    devServer = {
    	proxy: {
    		'/api': {	// 匹配所有以 '/api' 开头的请求路径，此名字可自定义
    			target: 'http://localhost:8081',  // 资源所在地的请求路径【PS：不加sourceName资源名 】
    			ws: true,	// 是否支持websocket 【PS：vue中默认值就是true，react中是false 】
    			changeOrigin: true,		// 代理服务器是否要伪装请求路径【PS：vue中默认值就是true，react中是false 】
    			pathRewrite: { '^/api': '' }	// 去掉请求路径中的 api 前缀名  如：ip:port/api/student
			},

            // 配置多套代理
            '/api2': {
    			target: 'http://localhost:8082',
    			ws: true,
    			changeOrigin: true,
    			pathRewrite: { '^/api2': '' }
			}
		}
	}
}
```

其中：
- target：就是资源所在的服务器地址 `http://ip:port`。
- ws：就是是否支持websocket。
- changeOrigin：就是代理服务器在请求资源地服务器时，代理服务器是否伪装路径

PS：假设资源地服务器地址是8081，而请求地是8080，则：此配置就是代理服务器在帮忙请求8081时，把自身伪装成8081，因为：有些服务器是有限定的，不是同路径不能访问资源。

- 如果要配置多套代理，那么：复制粘贴，改成对应的服务器即可。







2. 在代码中使用axios发起请求。

```javascript
axios.get / post('http://ip:port/preName/sourceName'),then(	// 其中：preName就是vue.config.js中配置的'/api'这个名字
    response => {
        console.log( response.data )
    },

    error => {
        console.log( error.message )
    }
)
```

其中：

- 加上preName  路径前缀名就是为了控制是否走代理服务器（为了解决前面本地中有资源时，默认获取本地资源的问题）。
- 不加preName  就是默认走本地，不走代理服务器。



<be/>



#### vue cli的两种跨域配置总结

第一种方式：

- 在`vue.config.js`中添加如下配置：




```javascript
devServer: {
    proxy: 'http://ip:port'  // ip和port为资源所在地的port
}
```

- 优点：配置简单，请求资源时直接发给发起请求的地方。

- 缺点：不能配置多个代理、不能灵活控制是否走代理。

- 工作方式：优先走本地【PS：发起请求的地方 】，没有要找的资源才去配置的服务器中找。







第二种方式：

- 在`vue.config.js`中添加如下配置：




```javascript
module.exports = {
    devServer = {
    	proxy: {
    		'/api': {	// 匹配所有以 '/api' 开头的请求路径，此名字可自定义
    			target: 'http://localhost:8081',  // 资源所所在地的请求路径【PS：不加sourceName资源名 】
    			ws: true,	// 是否支持websocket 【PS：vue中默认值就是true，react中是false 】
    			changeOrigin: true,		// 代理服务器是否要伪装请求路径【PS：vue中默认值就是true，react中是false 】
    			pathRewrite: { '^/api': '' }	// 去掉请求路径中的 api 前缀名  如：ip:port/api/student
			},

            // 配置多套代理
            '/api2': {
    			target: 'http://localhost:8082',
    			ws: true,
    			changeOrigin: true,
    			pathRewrite: { '^/api2': '' }
			}
		}
	}
}

```

- 使用axios发起请求即可




```javascript
axios.get / post('http://ip:port/preName/sourceName'),then(	// 其中：preName就是vue.config.js中配置的'/api这个名字
    response => {
        console.log( response.data )
    },

    error => {
        console.log( error.message )
    }
)
```



> 注意：这两种配置不能同时存在，因为它们都是占用了proxy这个对象【PS：二者都是配置在这里面的，写法不一样 】









### 插槽

基础代码：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202171640116-172569630.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202171708653-2037592212.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202171731755-2014293120.png)









#### 默认插槽

> 此种插槽适合只占用一个位置的时候。

需求、让食品分类中显示具体的一张食品图片、让电影分类中显示某一部具体的电影，使用默认插槽改造。

```html
<template>
  <div class="container">
    <Category title = "食品">
      <!-- 2、将内容套在组件标签里面从而携带到slot处 
              此种方式：是vue在解析完App这个组件的模板时，
                        将整个组件中的内容给解析完了，然后放到了Category组件里面使用slot占位处
                        所以：slot所放位置 和 这里面代码解析之后放过去的位置有关
              另外：由于是先解析完APP组件中的东西之后 再放到 所用组件里面slot处的位置
                    因此：这里可以使用css+js，这样就会让 模板 + 样式解析完了一起放过去
                          不用css+js就是先把模板解析完了放过去，然后找组件里面定义的css+js
      -->
      <img src="./assets/food.png" alt="照片开小差去了">
    </Category>

    <Category title = "游戏">
      <ul>
          <li v-for=" (game,index) in games" :key="index">{{game}}</li>
      </ul>
    </Category>

    <Category title = "电影">
      <video controls src="./assets/枕刀歌(7) -  山雨欲来.mp4"></video>
    </Category>
  </div>
</template>

<script>
import Category from "./components/Category.vue"
  export default {
    name: 'App',
    components: {Category},
    data() {
      return {
        foods: ['紫菜','奥尔良烤翅','各类慕斯','黑森林','布朗尼','提拉米苏','牛排','熟寿司'],
        games: ['王者荣耀','和平精英','英雄联盟','文明与征服','拳皇','QQ飞车','魔兽争霸'],
        filems: ['无间道','赤道','禁闭岛','唐人街探案1','肖申克的救赎','盗梦空间','无双']
      }
    },
  }
</script>

<style>
  .container {
    display: flex;
    justify-content: space-around;
  }

  img,video {
    width: 100%;
  }
</style>
```



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202193610857-1695745311.png)



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202192519266-1092967433.png)





#### 具名插槽

> 指的就是有具体名字的插槽而已，也就比默认插槽多了两步罢了。
>
> 在使用slot进行占位时利用name属性起一个名字，然后在传递结构时使用`slot="xxx"`属性指定把内容插入到哪个插槽就OK了。

需求、在电影分类的底部显示"热门"和"悬疑*。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202200719691-998706771.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202200808587-379268855.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202200900404-88105923.png)







#### 作用域插槽

> 这个玩意儿的玩法和具名插槽差不多，只是<span style = "color:blue">多了一点步骤、多了一个要求、以及解决的问题反一下</span>即可。

基础代码：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202205747825-112725568.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202205818173-1822187345.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202205836806-1121376602.png)





需求：多个组件一起使用，数据都是一样的，但是有如下要求：

- 数据显示时要求不是全都是无序列表，还可以用有序列表、h标签......
- 要求data数据是在子组件中【PS：就是Category中 】，而结构需要写在父组件中【PS：就是App中 】，也就是父组件要拿到子组件中的data。
- 利用前面已经玩过的子父组件通信方式可以做到，但是麻烦。因此：改造代码。



改造代码：

1. 第一步：提data。



![image](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215153037817-1807317389.png)

查看效果：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202210726359-33402565.png)

可能会想：明明可以将data放到父组件中 / 将结构ul放到子组件中，从而实现效果，为什么非要像上面那样折腾，没事找事干？开发中有时别人就不会把数据交给你，他只是给了你一条路，让你能够拿到就行。







2. 第二步：使用作用域插槽进行改造。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202212340997-1904612609.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202212422663-953283551.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202212640584-1456431441.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202213002034-1713986774.png)







既然作用域插槽会玩了，那就实现需求吧。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202213814199-158161312.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202214433935-61004651.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202214659737-1071231474.png)



> 另外：父组件接收数据时的scope还有一种写法，就是使用slot-scope

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220202215008861-2128780647.png)







#### 插槽总结

作用：让父组件可以向子组件指定位置插入HTML结构，也是一种组件间通信的方式，适用于：**父组件 ===》 子组件**。

分类：默认插槽、具名插槽、作用域插槽。

使用方式：

1. 默认插槽



```html
// 父组件
<Category>
    <div>
        HTML结构
    </div>
</Category>

// 子组件
<template>
    <div>
        <!-- 定义插槽 -->
        <slot>插槽默认内容</slot>
    </div>
</template>
```





2. 具名插槽



```html
// 父组件
<template>
    <!-- 指定使用哪个插槽 -->
    <Category slot = "footer">也可以加入另外的HTML结构</Category>
</template>



// 子组件
<template>
    <div>
        <!-- 定义插槽 并 起个名字-->
        <slot name = "footer">插槽默认内容</slot>
    </div>
</template>
```



3. 作用域插槽



```html
// 子组件
<template>
  <div class="category">
      <h3>{{title}}分类</h3>
      <!-- 作用域插槽
        1、子组件（ 传递数据 ） 
            提供一个路口，把父组件想要的数据传给它【PS：有点类似于props的思路，只是反过来了 】
            :filems中的filems就是提供的路，给父组件传想要的东西【PS：对象、数据都行 】
      -->
      <slot :filems = "filems">这是默认值</slot>
  </div>
</template>

<script>
    export default {
        name: 'Category',
        props: ['title'],
        data() { // 数据在子组件自身中
            return {
                filems: ['无间道','赤道','禁闭岛','唐人街探案1','肖申克的救赎','盗梦空间','无双']
            }
        },
    }
</script>


// 父组件
<template>
  <div class="container">
    <Category title = "电影">
      <!-- 
        2、父组件（ 接收数据 ）
          前面说的 多了一个要求   就是这里"必须用template标签套起来"
          怎么接收？使用scope="xxxx"属性   xxx就是接收到的数据，这个名字随便取
              这个名字不用和子组件中用的 :filems 这个filems这个名字保持一致，因：它接收的就是这里面传过来的东西
              但是：这个数据有点特殊，需要处理一下
       -->
       <template scope="receiveData">
         <!-- {{receiveData}} -->
         <!-- 拿到了数据，那"页面的结构就可以随插槽的使用者随便玩"了 -->
         <ul>
           <li v-for="(filem,index) in receiveData.filems" :key="index">{{filem}}</li>
         </ul>
       </template>
    </Category>

    <Category title = "电影">
      <!-- ES6中的"结构赋值"简化一下 -->
       <template scope="{filems}">
         <ol>
           <li v-for="(filem,index) in filems" :key="index">{{filem}}</li>
         </ol>
       </template>
    </Category>

    <Category title = "电影">
      <!-- ES6中的"结构赋值"简化一下 -->
       <template scope="{filems}">
          <h4 v-for="(filem,index) in filems" :key="index">{{filem}}</h4>
       </template>
    </Category>

    <!-- scope还有一种写法 使用slot-scope-->
    <Category title = "电影">
      <!-- ES6中的"结构赋值"简化一下 -->
       <template slot-scope="{filems}">
          <h4 v-for="(filem,index) in filems" :key="index">{{filem}}</h4>
       </template>
    </Category>
  </div>
</template>

<script>
import Category from "./components/Category.vue"
  export default {
    name: 'App',
    components: {Category},
  }
</script>
```







### 状态机 Vuex

> 概念：在Vue实例中集中式状态（数据，状态和数据是等价的）管理的一个插件。说得通俗一点就是：数据共享，**对多个组件间的同一个数据进行读/写**，不就是集中式管理了吗（对立的观点就是"分布式"，分布式是后端的人玩的）。

github地址：https://github.com/vuejs/vuex



什么时候用vuex？

1. 多个组件依赖同一状态（数据）【PS：两个帅哥，都想要同一个靓妹】。
2. 不同组件的行为 需要 变更同一状态【PS：洗脚城的两个妹子 都想要把 客户钱包中的money变到自己荷包中】。
3. 上面的内容用一句话概括：**多组件需要分享数据时就使用Vuex**。







#### Vuex 原理

首先这个东西在官网中有，但是：不全。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220203165952840-461258590.png)





所以：把官网的原图改一下。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220203225912948-1839688466.png)

上述内容只是对原理图有一个大概认识而已，接下来会通过代码逐步演示就懂了。







#### 搭建Vuex环境

1. 在项目中安装vuex。指令：`npm install vuex`。

2. 编写store。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220204171317661-1536614282.png)



3. 让store在任意组件中都可以拿到。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220204171452293-1784034367.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220204171633298-1127404162.png)

这样store就自然出现在其他组件身上了【PS：利用了vc和vm的内置关系 ，验证自行验证，在其他组件中输出this就可以看到了】。



> 小结：vuex环境搭建

1. 创建文件 `src/store/index.js`。



```javascript
// 引入vuex
import vuex from "vuex"

// 使用vuex —— 需要vue 所以引入vue
import Vue from "vue"
Vue.use(vuex)


// 创建store中的三个东西actions、mutations、state
const actions = {}

const mutations = {}

const state = {}

// 创建store ———— 和创建vue差不多的套路
export default new vuex.Store({
    // 传入配置项 ———— store是actions,mutations,state三者的管理者，所以配置项就是它们
    actions,    // 完整写法 actions:actions ，是对象嘛，所以可以简写
    mutations,state
})
```





2. 在 `main.js` 中配置store。



```javascript
import App from "./App.vue"
import Vue from "vue"

// 引入store
import store from "./store"     
// 由于起的名字是index，所以只写./store即可，这样默认是找index，没有这个index才报错

const vm = new Vue({
    render: h=>h(App),
    components: {App},
    // 让store能够被任意组件看到 ———— 加入到vm配置项中【PS：和全局事件总线很像 】
    store,
    template: `<App></App>`,
}).$mount('#app')
```



<br>

#### 简单玩一下Vuex的流程

对照原理图来看：

![image](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240215155100969-428226381.png)



1. 先把要操作的数据 / 共享数据放到state中。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205150138017-1986830192.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205151033911-1382904240.png)



2. 在组件中使用dispatch这个API把key-value传给actions。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205143437275-1996455907.png)

actions就是第一层处理【PS：服务员嘛 】。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205151108453-1229417994.png)





actions接收key-value【PS：要是有逻辑操作，也放在这里面，ajax也是 】。

- 这里就是调用commit这个API把key-value传给mutation，这个mutation才是真正做事的人【PS：后厨嘛 】。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205150612285-941622705.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205151306030-2095782564.png)





3. mutations接收key-value。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205151729994-844968725.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205152035995-728259993.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205152229457-605415604.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205152313124-596315587.png)



4. 查看开发者工具【PS：简单了解，自行玩一下 】。

另外：vuejs devtools开发工具版本不一样，则：页面布局也不一样，但是：功能是一样的。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205153241046-635616394.png)



当然：前面说过，直接在组件中调commit这个API从而去和mutations打交道，这种是可以的，适用于：不需要逻辑操作的过程，示例就自行玩了。

以上便是简单了解vuex，前面的例子看起来没什么用，但是vuex这个东西其实好用得很。





#### 认识 getters 配置项

这玩意儿就和data与computed的关系一样

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205160217161-556984058.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205160343463-571458858.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205160532495-1264823964.png)





##### 四个map方法

###### mapState 和 mapGetters

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205163255617-1436674883.png)



> 1、改造源代码 —— 使用计算属性实现。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205163620316-675669252.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205163636913-125760798.png)

但是：上面这种是我们自己去编写计算属性，从而做到的，而Vuex中已经提供了一个东西，来帮我们自动生成计算属性中的哪些东西，只需一句代码就搞定。



> 2、使用mapState改造获取state中的数据，从而生成计算属性。

- 1）、引入mapState。代码：`import {mapState} from "vuex"`。

- 2）、使用mapState【PS：对象写法 】。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205164429374-1626360959.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205164554601-1727458862.png)









> 2.1、数组写法【PS：推荐用的一种 】。

`...mapState({sum:'sum'})` 这里面的 `sum:'sum'` 这两个是一样的，**那么：一名多用**。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205164839577-1639301460.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205164900329-53847574.png)





> 3、使用mapGetters把getters中的东西生成为计算属性

- 1、引入mapGetters。代码：`import {mapState,mapGetters} from "vuex"`。

- 2、使用mapGetters【PS：和mapState一模一样 ，对象写法 】。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205165441355-1587843448.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205165604941-174348409.png)



> 3.1、数组写法

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205165730211-1909747347.png)





###### mapActions 和 mapMutations

会了mapState，那么其他的map方法也会了，差不多的，只是原理是调了不同的API罢了，当然：也会有注意点。

> mapState 和 mapGetters是生成在computed中的，而mapActions和mapMutations是生成在methods中的。

1. mapActions —— 调的API就是dispatch。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205171536218-1306532615.png)



> 1、使用mapActions改造：
>
> - 1、引入mapActions 代码： `import {mapActions} from 'vuex'`。
>
> - 2、使用mapActions【PS：对象写法 】 —— 准备调入坑中。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205172200685-1737428363.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205172244292-36369521.png)



原因：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205172726026-680041007.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205172754491-694363738.png)





> 1.1、数组写法 —— 一样的，函数名和actions中的名字一样【PS：一名多用 】

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205172950691-990240480.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205173008564-427037928.png)



> 2、mapMutations —— 这个和mapActions一模一样，只是调用的API是commit

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205173238075-538160715.png)

所以：此种方式不演示了，会了前面的三种中任意一种，也就会了这个。





#### 简单玩一下组件共享数据

1. 在state中再加一点共享数据。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205183430180-357680226.png)



2. 新增Person组件。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205184225729-1021745566.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205183548015-256050897.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205184259206-1504260299.png)



3. 共享数据：在Count组件中获取person，在person中获取Count【PS：此步不演示，会了前者后者就会了 】

操作如下：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205184819473-68622492.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205184949731-2138749871.png)



#### Vuex模块化开发

##### modules + namespac

这玩意儿就是把actions、mutations、state、getters按照功能点进行对象封装【PS：但是注意这里面需要使用`namespace:true`开启命名空间，否则：在下一步中会报错，不允许配置。

然后在 `new vuex.store` 中使用modules配置项将前面封装的对象配置进来。





1. 对于使用map那四个的方式时，操作如下：



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205192654042-845008412.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205193443963-1105463384.png)



2. 对于自己写计算属性和methods时，操作如下：



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205194026575-2080541400.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205194737366-1389812434.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220205194917783-250408597.png)



##### vuex模块化开发总结

- 目的：让代码更好维护，让多种数据分类，更加明确。



- 修改`store.js`




```javascript
const countAbout = {
    namespace: true,	// 开启命名空间
    state: {
        x:1
    },
    actions: {......},
    mutations: {......},
    getters: {
        bigSum( state ){
            return state.sum * 10
        }
    }
}


const personAbout = {
    namespace: true,
    state: {.....},
    actions: {......}
    getters: {.......}
}


const store = new vuex.store({
    modules: {
        countAbout,personAbout   // 触发对象简写形式
    }
})
```



- 开启命名空间后，组件中读取state数据。




```javascript
// 方式一【PS：自己写计算属性时 】
this.$store.state.personAbout.list

// 方式二【PS：直接让mapState生成计算属性时 】
...mapState('countAbout',['sum','school','subject'])
```



- 开启命名空间后，组件中读取getters数据。




```javascript
// 方式一
this.$store.getters['personAbout/firstPersonName']

// 方式二
...mapGetters('countAbout',['bigSum'])
```



- 开启命名空间后，组件中调用dispatch。




```javascript
// 方式一
this.$store.dispatch('personAbout/addPersonWang',person)

// 方式二
...mapActions('countAbout',{incrementOdd:'jiaOdd',incrementWait:'jiaWait'})
```



- 开启命名空间后，组件中调用commit。




```javascript
// 方式一
this.$store.commit('personAbout/ADD_PERSON',person)

// 方式二
...mapActions('countAbout',{incrementOdd:'JIA',incrementWait:'JIAN'})
```



### 路由器 router

#### 认识路由和路由器

路由 route：就是一组key-value的映射关系。
- key：就是网站中的哪个路径。
- value：就是function 或 component组件。
  - function：是因为后端路由（后端调用函数，对该路径的请求做响应处理）。
  - component：组件就不用多说了。



路由器 router：就是专门用来管理路由的【PS：理解的话，就参照生活中的那个路由器，它背后有很多插孔，然后可以链接到电视机，那个插孔就是key ，而链接的电视机就是value】。



在vue中，router路由器是一个插件库，所以需要使用`npm install vue-router`来进行安装，这个东西就是专门用来做单页面网站应用的。

所谓的单页面网站应用就是 SPA，即：只在一个页面中进行操作，路径地址发生改变即可，然后就把相应的东西展示到当前页面，不会发生新建标签页打开的情况。
- SPA 整个应用只有`一个完整的页面`、点击页面中的导航链接`不会刷新页面`，只会做页面的`局部刷新`、数据需要通过ajax请求获取。







#### 简单使用路由器

1. 准备工作：引入`bootstrap.css`



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220206205313893-934608797.png)



2. 开始玩路由器 router

- 1）、给项目安装路由 指令：`npm install vue-router`。

- 2）、在 `main.js` 中引入并使用路由器【PS：路由器是一个插件 】。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220206212358265-1191781968.png)



- 3）、编写组件



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220206211358189-165420140.png)



- 4）、配置路由器【PS：这也叫配置路由规则，就是key-value的形式，在前端中，key是路径，value是组件 】。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220206212021982-1129716317.png)



- 5）、把配置的路由规则引入到 `main.js` 中



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220206212756011-1532255105.png)





- 6）、在静态页面中使用路由【PS：需要记住两个标签 `<router-link ..... to = "路径名"></router-link>` 和 `<router-view></router-view>` ]。

1、`<router-link ..... to = "路径名"></router-link>`  是指：跳转。其中：路径名 就是 路由规则中配置的 path，参照a标签来理解，本质就是转成了a标签。

2、`<router-view></router-view>`  是指：视图显示。就是告知路由器 路由规则中配置的component应该显示在什么位置，和slot插槽一样，占位。

```html
<template>
  <div>
    <div class="row">
      <div class="col-xs-offset-2 col-xs-8">
        <div class="page-header"> <h2>Vue Router Demo </h2></div>
      </div>
    </div>
    <div class="row">
      <div class="col-xs-2 col-xs-offset-2">
        <div class="list-group">
          <router-link class="list-group-item" active-class="active" to="./about">About</router-link>
          <router-link class="list-group-item" active-class="active" to="./home">Home</router-link>
          <!--对照a标签 <a class="list-group-item active" href="./home.html">Home</a> -->
        </div>
      </div>
      <div class="col-xs-6">
        <div class="paner">
          <div class="paner-body">
            <router-view></router-view>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'App',
  }
</script>
```



- 7)、运行效果如下：



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220206214308417-407272126.png)



> 另外：从此处开始，可以先摸索饿了么的 [Element-ui 组件库](https://element.eleme.cn/#/zh-CN/component/changelog) 了，这是专门搭配Vue来做页面的网站，和Bootstrap一个性质。@紫邪情







> 小结：vue-router使用

1. 安装vue-router ，命令：`npm install vue-router`。

2. 在`main.js`中引入，指令：`import VueRouter from "vue-router"`。

3. 在`main.js`中应用vue-router插件，指令：`Vue.use(VueRouter)`。

4. `src/rooter/xxx.js`中编写router路由规则：

```javascript
// 引入路由器
import VueRouter from "vue-router"
// 引入需要进行跳转页面内容的组件
import About from "../components/About.vue"
import Home from "../components/Home.vue"

// 创建并暴露路由器
export default new VueRouter({
    routes: [   // 路由器管理的就是很多路由  所以：routes 是一个数组
        {   // 数组里面每个路由都是一个对象 它有key和value两个配置项【PS：还有其他的 】
            path: '/about',     // 就是key  也就是路径名，如：www.baidu.com/about这里的about
            component: About    // 就是value  也就是组件
        },{
            path: '/home',
            component: Home
        },
    ]
})
```

5. 把配置的路由规则绑定到Vue实例上（`main.js`）

```javascript
// 把4中router文件夹中配置的路由规则引入进来
import router from "4中的router文件夹"

new Vue((
	render: h->(App),
	router,		// 完整写法	router:router	前为配置项名字		后为引入的路由规则
)).$mount('#app')
```


6、在需要跳转的组件中实现切换（active-class 可配置高亮样式）

```html
<router-link class="list-group-item" active-class="active" to="./about">About</router-link>
<!--to	要跳转到哪个组件		上一步4中路由规则中配置的 path -->

<!--对照a标签 <a class="list-group-item active" href="./home.html">Home</a> -->
```



7、指定展示位置

```html
<!-- 告知路由器 路由规则中配置的component应该显示在什么位置 -->
<router-view></router-view>
```





#### 聊聊路由器的一些细节

1. 路由组件以后都放到page文件夹下（非绝对），而一般组件都放到components中。

2. 路由切换时，“隐藏”了的路由组件，默认是被销毁掉了，需要时再去重新挂载的【PS：示例自行通过beforeDestroy和mounted进行测试 】。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207131542001-444078933.png)



3. 每个组件都有自己的`$route`属性，里面存储着自己的路由信息。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207132139127-653860779.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207132227024-1466542004.png)



4. 整个应用只有一个router，可以通过组件的`$router`属性获取到【PS：验证自行把不同组件的这个东西绑定到window对象上，然后等路由组件挂载完毕了，拿到它们进行比对，答案是：false 】。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207132516665-982748899.png)





#### 多级路由（子菜单）

1. 在`src/page`下再新建两个路由组件。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207144032424-1512459721.png)



2. 给home路由规则编写多级路由。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207144256113-1910098938.png)



3. 重新编写`Hmoe.vue`路由组件。

```html
<template>
    <div>
        <h2>我是Home的内容</h2>
        <div>
            <ul class="nav nav-tabs">
                <li>
                    <!-- 多级路由，这里的to后面需要加上父级路径  先这么写，它可以简写，后续进行处理 -->
                    <router-link class="list-group-item" active-class="active" to="/home/news">News</router-link>
                </li>
                <li>
                    <router-link class="list-group-item" active-class="active" to="/home/message">Message</router-link>
                </li>
            </ul>
            <ul>
                <router-view></router-view>
            </ul>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'Home'
    }
</script>
```





4. 运行效果如下：



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207144639772-1489749752.png)





#### 路由传参

##### query的字符串写法：v-bind + 模板字符串

> 也就是路径传参，适合传递少量参数。

1）、编写数据，改造`Message.vue`路由组件【PS：传递数据者	v-bind + 模板字符串】。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207154808269-798424858.png)

2）、编写`Detail.vue`路由组件【PS：数据接收者	`$route.quesy.要取的参数名`】。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207154940154-360939790.png)

3）、编写路由规则。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207155021115-2090329054.png)

4）、效果如下：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207155055271-180909464.png)



##### query的对象写法：router-link to="quesy:{要传递的参数}"

> 适合传递大量参数。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207155549462-1000362538.png)

运行效果和上一节一样的。





#### 命名路由

> 这个玩意儿就是为了处理path中的那个字符串很长的问题，相当于起个别名。@紫邪情

实例：

1）、修改路由规则

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207163335896-1013984015.png)

2）、使用命名路由精简path

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207163438388-2022206240.png)

3）、运行效果如下

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207163517751-905077374.png)



> 小结：命名路由

作用：简化路由跳转时的path写法

使用：

1. 给命令命名

```json
{
       path: '/home',
       component: Home,
       children: [
           {
               path: 'news',
               component: News
           },{
               path: 'message',
               component: Message,
               children: [
                   {
                       path: 'detail',
                       // 使用另一个配置项name  命名路由，从而让path更精简
                       name: 'detail',
                       component: Detail
                   }
               ]
           },
       ]
   },
```

- 简化路由跳转写法：



```html
<!-- 简化前写法 -->
<router-link 
   :to="{
       path: '/home/message/detail',
       query: {
           id: m.id,
           title: m.title
       }
   }">
   {{m.title}}
 </router-link>

<!-- 简化后写法 -->
<router-link 
   :to="{
       name: 'detail',
       query: {
           id: m.id,
           title: m.title
       }
   }">
   {{m.title}}
 </router-link>
```





#### 路由另一种传参：params

> 注意：这种传参对象式写法必须基于name配置项才可以。
>
> 另外就是：这种传参也就是后端中的RESTful传参。

使用params传递参数【PS：数据的传递者 】，这一步和以前的query传递没什么两样，只是改了一个名字而已。

1）、字符串写法 / 对象写法。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207170611050-1440674896.png)

2）、修改路由规则。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207170834000-901260994.png)

3）、获取参数 【PS：和query相比，就是数据存放的位置变了一下而已，可以在mounted中输出`this.$route`看一下结构 】。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207170924769-344217251.png)

4）、效果如下：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207171047271-599393251.png)



> 小结：路由params传参

1、配置路由

```json
{
   path: '/home',
   component: Home,
   children: [
       {
           path: 'news',
           component: News
       },{
           path: 'message',
           component: Message,
           children: [
               {
                   // 使用params传参，则：需要把path的规则改了，就是占位，接对应参数
                   path: 'detail/:id/:title',
                   name: 'detail',     // 对象写法，必须保证有这个配置项
                   component: Detail
               }
           ]
       },
   ]
},    
```



2、传递参数

```html
 <!-- 使用params传递参数 -->

 <!-- 字符串写法 -->
 <router-link :to="`/home/message/detail/${m.id}/${m.title}`">
   {{m.title}}
 </router-link>

 <!-- 对象写法 
   这种写法必须保证里面是name，而不是params，否则：页面内容会丢失的
 -->
 <router-link 
   :to="{
       name: 'detail',
       params: {
           id: m.id,
           title: m.title
       }
   }">
   {{m.title}}
 </router-link>
```

- **注意点**：路由携带params参数时，若使用的to的对象写法，则：不能使用path配置项，必须用name配置项



3、接收参数

```javascript
{{$route.params.id}}
{{$route.params.title}}
```





#### 路由的props配置项

props这个东西在组件的父传子通信时见过，但是：不是一回事，那是组件间的，现在是路由的，写法几乎不一样。

> 这里有一句话：哪个路由组件要接收数据，props就配置在哪个路由规则。

回到问题：插值语法讲究的就是简单的模板语法，而下图这种就要不得，所以：要简化。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207180011871-466902099.png)



##### 布尔值写法

> 为true时，则把path接收到的所有params参数以props的形式发给所需路由组件。

注意：是params传递的参数，所以：这就是缺点之一。

另外就是：以props形式传给所需组件，所以：在所需路由组件那边需要使用`pros:['xxxx']`来进行接收，从而在所需组件中使用数据时就可以进行简化了。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207190649034-1182526190.png)





##### 函数写法

> 这种写法：是最灵活的一种，props为函数时，该函数返回的对象中每一组key-value都会通过props传给所需数据的路由组件。
>
> 
>
> 这种写法可以获取query传递的数据，也可以接收params传递的，注意点就是：在函数中调用时的名字变一下即可。

这种的好处就是：一是query和params都可以接收，二是：把数据接收的逻辑写到要接收数据的路由组件的路由规则去了，逻辑更清晰，不至于到处找逻辑代码。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207192032037-1004526527.png)



**另外的方式**：不推荐使用，所以：简单了解即可。

1）、在数据使用者处自行抽离代码，弄成计算属性 【PS：如示例中的Detail路由组件，取数据时在插值语法中麻烦，那就在下方配置计算属性从而抽离代码，但是：画蛇添足，因为：在计算属性中拿数据时会使用`this.$route.queru / params.xxxx`  代码量大的话，这不就还得多写N多this吗。



2）、对象写法 —— 不用了解，知道有这么一个东西即可，需要时自行百度即可【PS：这个东西接收数据是死的，开发中基本上用都不用 】。

- 此种方式：是将该对象中所有的key-value的组合最终通过props传给所需路由组件。







#### router-link中的 replace 属性

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207195919715-381939580.png)



上面这种模式就是push模式，它的原理就是栈空间，压栈push嘛【  **PS：router-link中的默认模式就是push模式** 】

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207200451123-333742717.png)

但是还有一种模式是：replace模式，这种模式是产生一个记录之后，就把上一次的记录给干掉了，所以效果就是不会有回退的记录，回退按钮都点不了要使用此种模式就是在router-link中加一个replace属性即可。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207201325526-509540147.png)

这样之后，再点击Home / About时，它的历史记录不会留下

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220207201437463-784995903.png)



> 小结：router-link的replace属性

- 作用：控制路由跳转时操作浏览器历史记录的模式。
- 浏览器的历史记录有两种写入方式，分别为`push`和`replace`，其中：`push`是追加历史记录，`replace`是替换当前记录，路由跳转时默认为`push`。
- 开启replace模式的方式：`<router-link replace ......>News</router-link>`。







#### 编程式路由导航

这玩意儿就是不再借助`router-link`来实现路由跳转，前面玩了`$route`，而现在就是来玩的`$router`。

先看一下`$router`这个东西，顺便知道掌握哪些API。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208173352226-768143022.png)





##### push 和 replace 两个API

> push和replace的原理就是前面说的，一个保留历史记录，一个会清除上一次的历史记录。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208174703155-1063972978.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208174750440-291671062.png)



2. 玩一下back和forward这两个API：这两个就是浏览器中的前进和后退



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208192655487-1923581115.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208192937002-258825793.png)



3、玩一下go这个API：`go()`中可以用正数和负数，正数表示：前进所填数字步；负数就是后退所填数字的绝对值步。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208193323838-274343377.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208193422419-1153183910.png)





#### 缓存路由组件

前面不是说了：路由组件在被切换走之后，是会被销毁的，因此：这种情况就会导致有时某个路由组件中的数据不应该被销毁，而是保留起来，所以：就需要借助即将说明的知识点，就是使用了一个`<keep-alive include = "componentName"></keep-alive>`标签来实现。



##### 字符串写法：缓存一个路由组件

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208194802899-1503203189.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208195003172-399333131.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208195424446-631249688.png)

方便验证，加上如下的代码：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208195654593-2003005560.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208200110334-1943501978.png)





##### 数组写法：缓存多个路由组件

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208201817430-364856722.png)

演示就不做了。







### 另外的生命钩子

在基础篇中就说过：除了哪里讲的8个生命钩子【PS：4对 】，其实还有三个生命钩子没有玩，也说了等到路由之后再整。



#### activated 和 deactivated

> activated：就是激活  【PS：我想见你了，就调它 】。
>
> deactivated：就是失活  【PS：我不想见你了，你离开吧，就调它，有点类似于beforeDestroy这个钩子，但是：处理的情况不一样 】。
>
> 适用场景：提前使用了`keep-alive include = "xxx"`保留该组件，切换后不让其销毁【PS：beforeDestroy不起作用了 】，那么：又想要最后关掉定时器之类的，就可以使用这两个钩子函数。

实例：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208212225059-177083237.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208212548472-2048172954.png)





#### nextTick

这个东西不演示了，**它是为了：让解析时产生时间差**。

因为：有些东西需要把解析完之后的样子插入到页面中了才可以继续做另外的事情，因此：就可以借助nextTick，从而：让一部分模板先被解析好，放入页面中，然后再解析后面的一些东西时执行一些我们想要的逻辑。如：input框。有这么一个场景：让input渲染到页面时，我鼠标的焦点就在input框中，这就可以使用此钩子函数，在挂载时调用此钩子，然后把光标聚焦的逻辑放到nextTick回调中。



nextTick钩子的玩法官网有，一看就懂。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208213220952-1103550446.png)





### 路由守卫  - 重要

> 所谓的路由守卫，就是权限问题，即：拥有什么权限，才可以访问什么路由。



#### 全局前置路由守卫

> 所谓的全局前置路由守卫，特点之一就体现在前置二字上，它的API是 `beforeEach((to, from, next) => {})`，也就是：在路由切换之前回调会被调用 / 初始化渲染时回调会被调用 。



实例：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208222734918-275765763.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208224009409-1025598036.png)

不方便演示，所以直接说：

- to：就是切换路由组件之后的组件位置【PS：即 去哪里，目标组件 】；
- from：是 切换路由组件之前的组件位置 【PS：即 来自哪里 从哪个路由组件来 】；
- next：是一个函数`next()`，就是放行的意思。



现在做一个操作，在浏览器中缓存一个key-value，然后访问News、Message路由时判断key-value是否对得上，对就展示相应的路由组件，否则：不展示。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208224754448-84407836.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208230401678-1575481065.png)



玩点小动作

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208230516743-1770304126.png)





当然：前面的过程有一个小技巧可以简化

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208232512869-569499621.png)

给路由规则添加一个meta配置项，就是路由元数据，利用这个配置项，我们可以给路由中放一些我们想放的东西进去【PS：哪个路由需要权限验证，就在哪个路由中加入meta配置项 】，那就改造吧。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208233133432-1556557448.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208233523579-495163945.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220208233627393-669008741.png)





#### 全局后置路由守卫

这玩意儿就和全局前置路由守卫反着的嘛，但是：处理场景不一样。

> 全局后置路由守卫 调用的API是 `afterEach((to, from) => {})`。这个API是 初始化渲染时回调会被调用 / 路由组件切换之后会被调用。
>
> 注意：和全局前置路由守卫相比，少了next参数，后置了嘛，都已经在前面把权限判断完了，你还考虑放不放行干嘛。

这个全局后置路由守卫做的事情其实不是去判断权限问题，而是收尾，做一些过了权限之后的判断问题，比如：点击某个路由组件之后，只要可以查看这个组件内容，那么：就把网页的页签标题给换了。



实例：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209000034040-136137973.png)

操练一手：【PS： 先用纯的全局前置路由守卫来做 】。

先加点东西进去：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209000541947-1069677415.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209001225857-973487877.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209001353001-222460191.png)



上面看起来成功了，但是有bug，可以试着把network中的网络调成slow 3G，可以稍微清楚地看到效果 ，想要改成功，就算把项目中 public/index.html的title改了，也是一样，会有加载过程，因此：想在全局前置路由守卫中达到想要的效果，改出花儿来也莫得办法，而且在全局前置路由守卫中写两遍一样的代码根本不标准。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209001652020-361327246.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209001917235-931711342.png)



想要实现前面的效果，那就需要全局后置路由守卫登场了，掉一下API，里面一行代码搞定。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209002619390-1052859904.png)

效果就不演示了，已经达到效果了【PS：注意得把 `public/index.html` 中的title改成'大数据智慧云生平台'，不然访问根目录时也有加载过程，这不是此知识点的锅，因为：原生的 `public/index.html` 的title是读取的 `package.json` 中第二行的name 】。





#### 独享路由守卫

> 这玩意儿就是指：某一个路由独享的路由守卫，调用的API是：`beforeEnter( ( to, from, next )=>{ } )`，它是指：在进入配置这个API的路由之前回调会被调用，其中：to、from、next的意思和前面全局前置路由守卫一样。
>
> 但注意：这种没有什么后置之类的，它只有这一个前置，即：独享路由守卫。
>
> 另外：路由守卫之间，是可以随意搭配的。

实例：

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209004839973-1076969366.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209005023294-583302194.png)







#### 组件内路由守卫 - 了解

> 这种路由守卫就是组件内的，它使用的API是` beforeRouteEnter( ( to, from, next )=>{ } )` 和 `beforeRouteLeave( ( to, from, next ) =>{ } )`。

```javascript
beforeRouteEnter(( to, from, next )=>{ })		指的是：通过路由规则，进入配置了这个API的路由组件之前回调会被调用

beforeRouteLeave(( to, from, next )=>{ })		指的是：通过路由规则，离开配置了这个API的路由组件之前回调会被调用
```

> 注意：是通过路由规则进入 / 离开这个API所在的路由组件，切记是通过路由规则做到的，因为想要路由组件工作，不用经过路由规则也可以让其工作【PS：就是以前组件的正常使用流程，引入、注册、使用组件标签 】。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209134503958-501739188.png)



### 路由器的两种工作模式：hash 和 history

1. **hash模式  路由器的默认模式**。

就是路径中有一个`#`，这`#`后面的内容不用随http传给服务器，服务器也不会收到【PS：前端玩一下而已 】。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209134739309-82655158.png)



2. **history模式**

这个就好理解了嘛，就是没有了那个#，然后路径中`ip:port`之后的东西是会随着http传给服务器的。





> hash和history两种模式的区别：

1. hash模式 路径中有`#`，且#后的内容不会随http传给服务器；而history模式 路径中没有`#`，`ip:port`之后的东西会随http传给服务器。
2. hash模式的兼容性好，而history模式的兼容性略差。





> 在路由器中hash和history两种模式的切换：在路由规则中加个全新的配置项mode即可。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209140036086-1609197708.png)





### 关于项目上线的问题

> 了解这个东西是因为前面说的hash和history的另一个区别，在上线时有个坑 / 注意点。

1. 编写完了程序之后打包项目。

启动项目一直用的是`npm run serve`，在脚手架时就说过还有一个命令：`npm run build`，那时说过：后端要的前面资源是HTML+CSS+JS，所以此时项目打包就需要用到它了。

先把路由器的工作模式切换成history模式，然后再打包，这样方便演示bug。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209144528552-543002828.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209145121595-1234579740.png)



2. 使用node+express框架编写一台小服务器模拟一下上线。

自行新建一个文件夹，然后使用vscode打开。

1）、让文件夹变成合法包 指令：`npm init`。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209150259175-1663272756.png)

2)、安装express  指令：`npm install express`  

**注意**：这一步很容易因为自己当初配置nodejs时操作不当，导致权限不够啊，就会报一堆warn和error。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209150655650-1495842035.png)

3）、新建一个js文件，编写内容如下

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209151808266-1193601529.png)

源码如下：

```javascript
// 1、引入express  注意：这里就不是ES6的模块化了，而是commanjs模块化
const express = require('express')

// 2、创建一个app服务实例对象
const app = express()

// 3、端口号监听
app.listen(8001,(err)=>{    // err是一个错误对象
    if( !err ) console.log("服务器启动成功");
})

// 4、配置一个后端路由
app.get('/person',(req,res)=>{      // req就是request res就是response
    res.send({
        name: '紫邪情',
        age: 18
    })
})
```



4)、启动服务器 指令：`node server`

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209151932015-1708156484.png)

5)、访问服务器中的端口测试一下

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209153045124-631944585.png)



3. 准备工作弄完了，现在把刚刚使用`npm run build`打包的dist中的文件复制到服务器中去。

在服务器中新建一个static / public文件夹【PS：后端的人，这两个文件夹就是SpringBoot中的那两个，这两个文件夹建哪一个都可以 】。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209152613499-980681026.png)



4. 让复制进去的文件能够被服务器认识。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209153827638-345035535.png)

重新执行`node server` 开始演示路由器的两种工作模式的另一个坑【PS：别忘记有个权限认证啊，在缓存把对应东西放上，不然有些路由组件点不了 】

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209153946746-1663995128.png)

整bug，随便点一些路由组件之后，刷新页面【PS：只要保证路径不是 `localhost:8001` 即可，让它后面有点东西 】。

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209154203840-266553940.png)



> 这就是history模式的坑（切换成hash就不会出现这样）。
>
> 出现上述的情况是因为：刚刚我们在页面中随便点路由组件都有页面是因为：那些都是静态页面，那些数据啊、历史记录啊都是原本就有的，即：不走网络请求，但是：刷新之后，是走网络请求的，也就会把路径中`ip:port`之后的东西随着http发给服务器了，它去服务器找资源就是找`http://localhost:8001/home/news`中的 /home/news，服务器中哪有这个资源，所以：404呗。

想要history模式也和hash一样，刷新不出错，就需要找后端人员进行处理，需要后端人员配合你这边拿过去的资源来做，后端处理这种问题的方式有很多，如：Nginx、跨域配置、通过注解.....【PS：但是嘛，有时别人甩你个锤子，最终代码出问题是自己背锅罢了】，所以：自己解决，需要借助一个插件 connect-history-api-fallback

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209155056476-1993512310.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209155321503-715720575.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209160006913-2077755972.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209160414018-259926824.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209160539517-249863598.png)



### Vue UI组件库 - Element-UI

使用：

- 1）、给项目安装Element-UI 指令：`npm i element-ui`。

- 2)、在项目的`main.js`中引入Element-UI 和 其样式。



```javascript
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
```



- 3)、在main.js中使用element-ui 指令：`Vue.use(ElementUI);`



但是：直接通过引入Element-UI和其样式，然后通过`Vue.use( xxxx )`来使用Element-UI会造成项目中的Element-UI的js体积太大了，不好，所以：**按需引入**。





#### 按需引入

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209195028892-1918956246.png)



1. 修改`babel.config.js`文件内容。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209195116773-2100577538.png)



2. 引入自己想要的组件和样式。



![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209195459559-1296465324.png)

> 注意：后续遇到的坑，还是需要在`main.js`中把全部样式引入，命令：`import 'element-ui/lib/theme-chalk/index.css';`

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209200504160-1139625236.png)

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209200434422-440110571.png)

当然：也可能报`not found xxx`，这种情况：直接`npm install xxx`即可，即：缺啥拉啥。

**解决上面报的问题：**

![image](https://img2022.cnblogs.com/blog/2421736/202202/2421736-20220209200249114-1683260585.png)





