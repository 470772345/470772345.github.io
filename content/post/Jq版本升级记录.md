---
title: "都2019了,你还在使用jQuery吗?"
date: 2020-05-12T00:24:31+08:00
draft: false
---

### 都2019了,你还在使用jQuery吗???
BTW,出来打工卖身,公司的老系统,安排你来维护,你还是要维护,谁让你是无处不能的程序猿呢(哈哈哈,先吹一波~)

所以先来分析一波当前的情况.
其实做加功能什么的还好说,这次是把项目中的jQuery的版本升级,v1.8.2升级到2.0+,因为上级说会安全点.好吧,拿项目是怎么样的呢,
系统采用 **Bootstrap + jQuery** 搭建开发的后台管理系统,涉及页面**100多个**.我一个人升级.所用时间要尽量的短.

> #### 框架升级大概会涉及**主要**问题:
> - [x] 涉及页面很多,但有些可以用**全局替换**,有些因语法差异大,不能一步到位
> - [x] 语法调整,事件弃用,这个要**注意替换的方法会不会对之前逻辑业务有影响**
> - [ ] UI框架有变,到时页面混乱,还好这次没有遇到

老系统的业务,也就看过几个页面.完全可以说是完全不了解,没有接触过来形容.
那怎么快速了解大概业务和相关模块功能呢??? 而且我还要统计页面的出错情况,毕竟升级后,影响程度有3种,1.当前页面大多被影响2.当前页面少数被影响3.当前页面无影响

放大招了~ ***直接下载工具,制作思维导图来记录***,在制作的过程还可以了解哪些模块以及功能.

***如图(部分截图):从系统出发,记录每一个子模块.记录出错,能大概推断出原因的写出原因***  

工具我使用是xmind.

![](https://user-gold-cdn.xitu.io/2019/8/22/16cb73cd6f1f82d5?w=694&h=700&f=png&s=53741)

以下是我遇到一些问题

## 判断浏览器类型 兼容
```
function userBrowser(){  
    var browserName=navigator.userAgent.toLowerCase();  
    if(/msie/i.test(browserName) && !/opera/.test(browserName)){  
        alert("IE");  
        return ;  
    }else if(/firefox/i.test(browserName)){  
        alert("Firefox");  
        return ;  
    }else if(/chrome/i.test(browserName) && /webkit/i.test(browserName) && /mozilla/i.test(browserName)){  
        alert("Chrome");  
        return ;  
    }else if(/opera/i.test(browserName)){  
        alert("Opera");  
        return ;  
    }else if(/webkit/i.test(browserName) &&!(/chrome/i.test(browserName) && /webkit/i.test(browserName) && /mozilla/i.test(browserName))){  
        alert("Safari");  
        return ;  
    }else{  
        alert("unKnow");  
    }  
}  
```
## 判断浏览器版本
```
var br=navigator.userAgent.toLowerCase();  
var browserVer=(br.match(/.+(?:rv|it|ra|ie)[\/: ]([\d.]+)/) || [0, '0'])[1]; 
```

```
$(selector).live(events, data, handler);                // jQuery 1.3+
$(document).delegate(selector, events, data, handler);  // jQuery 1.4.3+
$(document).on(events, selector, data, handler);        // jQuery 1.7+
```

### 升级后, 点击第一次全选生效,第二次开始不生效,换成 prop 解决bug

```
 $("#selectAll").change(function () {
        // if ($(this).attr("checked") == "checked") {
        if ($('#selectAll').is(':checked')) {
       //  $("input[name='checkList']").attr("checked", true);  jq1.8.3 ->jq2.0+ 的时候  
       //  点击第一次全选生效,第二次开始不生效,换成 prop 解决bug
            $("input[name='checkList']").prop("checked", true);
        } else {
            // $("input[name='checkList']").removeAttr("checked");
            $("input[name='checkList']").attr("checked", false);
        }
        // 下面 可以用来记录 点击的 次数
        arguments.callee.num = arguments.callee.num ? arguments.callee.num : 0;
        arguments.callee.num++
        console.log( arguments.callee.num)
    });
```
### 其中arguments.callee的使用
> argument为函数内部对象，包含传入函数的所有参数，arguments.callee代表函数名，多用于递归调用，防止函数执行与函数名紧紧耦合的现象，对于没有函数名的匿名函数也非常起作用。举例如下：

```
function factorial(num){
      if(num<=1){
          return 1;
      }else{
          return num*arguments.callee(num-1);  //arguments.callee代表factorial
      }
  }
  var trueFactorial = factorial;
  factorial = function(){
      return 0;
  }
   alert(trueFactorial(5)); //结果为120，因为js中函数没有重载，所以如果递归调用时使用函数名，则执行最后一个该函数名的函数，即返回0
   alert(factorial(5));//结果为0
```

匿名函数递归
```
var num = (function(num){
      if(num<=1){
          return 1;
      }else{
          console.log(num)
          return num*arguments.callee(num-1);
      }
 })(5);
  alert(num); //结果为120
```

### $('#id').live('event',fn) 弃用问题

> 我是使用 $(document).on('event','selecto',fn) 替换的


还有其他问题陆续填坑中.......

相关链接
https://www.html.cn/jqapi-1.9/live/