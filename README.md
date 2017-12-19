# RxJS  

##  简介
- 名字的由来：Reactive Extension  
- 源自微软、火于NetFlix, 是angular官方重要依赖库之一。  
- 在思考的维度上加入时间考量，具有跨平台的多语言支持的。 
- 官方网站：http://reactivex.io/rxjs/  
- 实例讲解  
(1)实例环境: JSBIN   
地址:   
https://jsbin.com/zulotomeyi/edit?html  
(2)引用外部地址：  
https://unpkg.com/@reactivex/rxjs@5.0.3/dist/global/Rx.js  
(3)选用的脚本语言是 ：ES6  

界面如下图所示:

JS代码:
1. 　  获取id为height的dom对象
1. 　  将height的keyup事件转换为Rx的事件流, 即Observable, 
1. 变量命名以$结尾，这是一种约定俗称的用法，Rxjs的数据
1. 是以流的形式存在，以$结尾表明这是一个数据流，即string。
1. 　 Rx就是一个观察者模式, 因此可以订阅
 　 

```
const height = document.getElementById("height");
const height$ = Rx.Observable.fromEvent(height, "keyup");
height$.subscribe( val => console.log(val))
```
控制台输出:

图 1-2 控制台输出图  
height$的订阅函数里面输出了height的keyup的事件对象。
获取height的value值 ，即代码改造


```
height$.subscribe( val => console.log(val.target.value + ' ' + new Date()))
```
Console的控制台输出结果:
图 1-3 控制台输出图
在文本框内陆续输入123456, 控制台输出结果。  
Rxjs 在控制台上输出一系列数据，这些数据在不同的时间点上，形成了沿着不同时间点走的事件流，即数据流。在不同的时间点上发射出一个值，这些值汇总成了Rxjs的数据流。从结果上我们可以得出Rxjs对数据的处理上，是加入了时间的考量。
- Rxjs 如何合并和转换数据流
   Rxjs 除了在不同的时间点上发射出值之外，还能对多个数据流进行转换和合并，例如:
  Html 代码:

```
<div><input type="text" id="length"></div>
<div><input type="text" id="width"></div>
<div id="area"></div>
```
JS 代码：

```
const length = document.getElementById("length");
const length$ = Rx.Observable.fromEvent(length, "keyup").pluck("target", "value");
const width = document.getElementById("width");
const width$ = Rx.Observable.fromEvent(width, "keyup").pluck("target", "value");
const area = document.getElementById("area");
const area$ = Rx.Observable.combineLatest(length$, width$, (l, w) => {return l * w;})
area$.subscribe(val => area.innerHTML = val);
```
输出结果：
combineLatest操作符将length$和width$两个数据流进行合并依据条件转换成流area$。只有当两个流中都有数据的时候,才进行合并转换，如果其中一个流没有值，则不进行转换。
转换过程 如下图所示：  
当length$ 数据流中出现1 时，它会去width$数据流中，寻找距离时间点最近的值，直到 2 出现, rxjs将两个数据流进行合并转换,计算出结果2，当再次在length 输入3 时,它会去width$流中，寻找距离时间点出现最近的值，即2 ，rxjs 再次进行合并转换，计算出结果6。同样的道理，如果在width 输入值，也会去length$流中寻找距离时间最近的值，再次进行合并转换。  

将combineLatest换成zip操作符时，zip操作符也可将多个流进行合并，与combineLatest 不同的是，要求合并流必须有新值出现的时候，且必须是成对出现的时候，才进行合并。  
转换过程，如下图所示:


**事件流
   理解Rx的关键是要把任何变化都想象成流,Rx会把数据主动推送给你，因此只需要在subscribe里面监听就可以**。
##  创建类操作符

```
- from : 可以把数组、Promise、以及Iterable转化为Observable
代码示例：  
Js：  
const height = document.getElementById("number");
const height$ = Rx.Observable.fromEvent(height, "keyup").
map(ev => 3);
const width$ = Rx.Observable.from([1, 2, 3, 4]);
const area$ = Rx.Observable.combineLatest(height$, width$, (l, w) => l* w);

height$.subscribe( val => console.log(val))
width$.subscribe( val => console.log(val))
area$.subscribe( val => console.log(val));
```
输出结果:
height$流中的数据是常量3,  
width$流快速的发射出一系列的值，如下图所示
最后width$流中的值最后是4，最后合并后的新流中aera$流中的数据始终是12
-  fromEvent: 可以把事件转化为Observable
-  of: 接受一系列的数据，并把它们emit出去of与from 的不同之处，除了能接受数组之外，还能接受多个对象，作为数据发射出去  
##### Js代码： 

```
const height = document.getElementById("number");
const height$ = Rx.Observable.fromEvent(height, "keyup").
map(ev => 3);
const width$ = Rx.Observable.of({id:1, value:20}, {id:2, value:30});
const area$ = Rx.Observable.combineLatest(height$, width$, (l, w) => l* w.value);

height$.subscribe( val => console.log(val))
width$.subscribe( val => console.log(val))
area$.subscribe( val => console.log(val));
```
Ps: 创建类可以将传统的编程转换成响应式编程，是传统和响应式的一个过渡门槛

转换类操作符
- Map  将流中的数据转换成另外的数据
- mapTo 设置常量  
   操作场景：不关心流中的值，只关心这个流有没有发生，或者记录发成的次数
- pluck 是map 是一种简化，获取数据流对象上的某个属性值
##  Observable 的性质
- 三种状态： next 、error 、complete   
   next : 流的下一个状态  
   error : 捕获异常  
   complete : 全部完成  
   这是observable的三种状态，也是subscribe的三个参数
- 特殊的: 永不结束、Never、Empty(结束但不发射)、Throw
Never 和 Empty 一般用来做测试，创建某个指定的极限环境，流里面的值一般是空的
- 常见的工具操作符 do （调试）
##### Js 代码：


```
const interval$ = Rx.Observable.interval(100)
   .map( val => val * 2 )
   .do(v => console.log( ' val is ' + v))
   .take(3);
 interval$.subscribe(
    val => {
  console.log(val);
 },
   err => {
  console.log(err);
 },
  () => {
    console.log(" I am completed");
  }
)
```

do操作符在map之后，take之前, 在rxjs流中起到了一个临时中间站的作用, 在这里不能改变数据流中的值, 但是可以根据流中的数据，与外围事物进行交互，或者进行调试
- 变换类操作符 scan 
#####  Js 代码:


```
const interval$ = Rx.Observable.interval(100)
  .filter( val => val % 2 === 0)
  .do(v => {
    v = v + 3;
    console.log( ' val is ' + v);
  })
  .scan((x, y) => {
    //x  累加器
    //y 数据流发射出来的值
    return x+y;
  }).take(4)
interval$.subscribe(
  val => {
  console.log(val);
},
err => {
  console.log(err);
},
  () => {

    console.log(" I am completed");
  }
)
```



原理如下图所示：

first数据流 发射出来的值 是原始的值,   
Second 数据流是filter过滤之后的数据流   
Scan 函数有两个参数，x 是 累加器, y 是 流中发射出来的值，x默认值是0  
Three 数据流 ，因为x的默认值是0，数据流发射出来的值 即y 是0， 所以第一个点发射出来的值是0，x变为0。第二次second数据流中发射出来的值是2，即y是2, x+y=2 ,three 流第二次发射出来的值是 2, 第三次second数据流中发射出来的值是4，此时累加器x变成上次累加的结果2, x+y=6. three流第三次发射出来的值是6
- 数学类操作符 reduce
#####  Js 代码：


```
const interval$ = Rx.Observable.interval(100)
    .filter( val => val % 2 === 0)
    .do(v => {
      v = v + 3;
      console.log( ' val is ' + v);
    })
    .take(4)
    .reduce((x, y) => {
      return [...x, y];
    }, {})
  interval$.subscribe(
    val => {
     console.log(val);
  },
  err => {
    console.log(err);
  },
  () => {

    console.log(" I am completed");
  }
)
```


Reduce 并不进行数学运算，而是返回一个集合, 将流中的数据变成一个集合返回。第二个参数，可以作为一个数组集合返回，也可以作为一个字典对象返回  
- 过滤类操作符filter、take、first/last、skip
- 创建类操作符 Interval ,  Timer
#####   Js 代码示例:


```
const interval$ = Rx.Observable.interval(100).take(3);
interval$.subscribe(
  val => {
  console.log(val);
},
err => {
  console.log(err);
},
  () => {

    console.log(" I am completed");
  }
)
//指定开始的延迟时间和间隔时间，
const timer$ = Rx.Observable.timer(100, 100);
timer$.subscribe( val => {
  console.log(val);
});
```


过滤类操作符
- debounce  debounceTime 

- debounce  类似于一个滤波器, 将符合一定时间间隔或者规则的数据过滤出来  
debounceTime  将符合一定时间间隔的值。发射出来
    
-  Distinct  distinckUntilChanged
distinct 保证整个数据流中不能出现相同的数值，
distinckUntilChanged  保证数据与邻近的数据不能相同  
## 合并类操作符
- merge 是将两个流，严格按照时间的顺序，有序的交叉在一起
         不更改两个流中的任何东西
- concat 需要等待一个流完成之后，然后将另外一个流合并上
- combineLatest 需要
- zip 需要对齐现象
- withLatestFrom 以源事件流为基准  
   是将一个流作为主流，只有当这个流有新值发生时，才会去邻近的流去寻找最新发射出来的值。
##### Js代码：


```
const length = document.getElementById("length");
const length$ = Rx.Observable.fromEvent(length, "keyup").pluck("target", "value");
const width = document.getElementById("width");
const width$ = Rx.Observable.fromEvent(width, "keyup").pluck("target", "value");
const area = document.getElementById("area");
const area$ = length$.withLatestFrom(width$)
area$.subscribe(val => console.log(val));
```


高阶操作符
 拍扁的作用, 将多个数据流合并成一个流  
 流中的每个元素又是一个流，将多个流合并成一个流，多层合并一层，将流扁平化  
###  mergeMap ( flatMap) 
 会保留所有的订阅，会保证所有的外层元素所对应子流的订阅  

```
const length = document.getElementById("length");
const length$ = Rx.Observable.fromEvent(length, "keyup")
         .pluck("target", "value")
         .mergeMap(_ => {
           Rx.Observable.interval(1000)
         });
length$.subscribe(val => console.log(val));
```

###  switchMap  
  一旦有新外层元素进来，就会抛弃原来外层元素订阅的子流  

```
const length = document.getElementById("length");
const length$ = Rx.Observable.fromEvent(length, "keyup")
         .pluck("target", "value")
         .switchMap(_ => {
           Rx.Observable.interval(1000)
         });
length$.subscribe(val => console.log(val));
```

应用场景：
 删除文章，也要删除文章的评论。  
 正在删除一篇文章，如果突然来了另外一篇文章需要删除，如果需要继续删除原先的文章评论，则就是mergeMap，不需要处理原来文章的评论，则就是switchMap  
##  Observable的冷与热
- 热： 无论订阅的早晚, 订阅者都会得到相同的东西 类似于视频直博
- 冷： 只要开始订阅，数据流动都会从新开始  类似于视频点播
冷的示例代码：
##### Js代码：


```
const count$ = Rx.Observable.interval(1000);
const sub1 = count$.subscribe(val => console.log(val));
setTimeout(function() {
  const sub2 =  count$.subscribe(val => console.log(val));
}, 2000)
```

输出结果：

##### 热的示例代码：


```
const count$ = Rx.Observable.interval(1000).share();
const sub1 = count$.subscribe(val => console.log(val));
setTimeout(function() {
  const sub2 =  count$.subscribe(val => console.log(val));
}, 2000)
```


输出结果：  

- Subject 既是观察者，又是被观察对象，即可作为数据流的输出一方，又可作为流的观察者一方  
##### 1、subject   
##### js 代码：


```
const counter$ = Rx.Observable.interval(1000).take(5);
const subject = new Rx.Subject();
const observer1 = {
  next: (val) => console.log("1 :" + val),
  error: (err) => console.error(" ERROR >> 1" + err),
  complete: () => console.log(" 1 is completed"),
};
const observer2 = {
  next: (val) => console.log("2 :" + val),
  error: (err) => console.error(" ERROR >> 2" + err),
  complete: () => console.log(" 2 is completed"),
};

subject.subscribe(observer1);
// 往流里面推送两个数据
subject.next(10);
subject.next(11);
setTimeout(function() {
  subject.subscribe(observer2);
}, 2000);
counter$.subscribe(subject);
```


输出结果：

Subject是 热的Observable  
##### 2、ReplaySubject 
重播指定的个数的值  
输出的结果  

2 是从0,11开始重播，重播最新的前两个数值  
##### 3、BehaviorSubject 
重播最新的值

2是从0开始订阅，开始从最新的一个数值开始订阅
##### Angular 中的Rx支持 
大量内置Observable 支持： 如 Http， ReactiveForms 、Route 等  
Async Pipe ：在页面直接表示 {{ auto$ | async}} 不需要取消订阅

