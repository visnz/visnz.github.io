---
title: "Nodejs接入C++扩展"
date: 2018-02-17
type: ["应用"]
weight: 7
tags: ["计算机","NODEJS","c++"]
categories: ["运维"]
description: "c++结合V8模块扩展nodejs功能，用c++处理复杂计算"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/oldicon/V8.jpg"
---
我团队在年底举办的青年实践活动“粿奔计划”中，线下游戏RPG采用Nodejs+cpp+html完成业务逻辑。

因为V8引擎由c++完成，可以在cpp中直接引入``<v8.h>``，通过``node-gyp``与``building.gyp``结构文件渲染成一个node模块。cpp作为游戏引擎部分，保留可以被调用的js模块接口，在nodejs中引入模块并按平常调用。

## 书写cc（cpp）文件
新建一个文件，这里用的是``test.cc``
```cpp

#include <node.h>
#include <v8.h>
#include <string>
using namespace v8;
using namespace std;

void hello(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = Isolate::GetCurrent();
  //统一开头 获取js脚本上下文

  int value = args[0]->NumberValue() + args[1]->NumberValue();
  //将第一二个参数的数值相加
  Local<Number> num = Number::New(isolate, value);
  //将相加的value转化为可返回的数据类型

  String::Utf8Value param1(args[0]->ToString());
  String::Utf8Value param2(args[1]->ToString());
  //新建String对象，由参数强制转换为String而填充
  string ret=string(*param1).string::append(string(*param2));
  //以上完成简单的字符转换和衔接
  Handle<Value> str = String::NewFromUtf8(isolate, ret.c_str() );
  //通过ret.c_str()转化对象后构建新Handle（handle 是指向对象的指针


  Local<String> str2= String::NewFromUtf8(isolate,"sss");
  //创建一个可以用于返回的String类型，内容为“sss”

  args.GetReturnValue().Set(str);
  //获取传入参数的返回参数，设置str为返回参数

  //如果传入的参数携带回调函数，则支持：
  // callback, 使用Cast方法来转换
  Local<Function> callback = Local<Function>::Cast(args[1]);
  Local<Value> argv[1] = {
    // 拼接String
    String::Concat(Local<String>::Cast(args[0]), String::NewFromUtf8(isolate, " world"))
  };
  // 调用回调, 参数: 当前上下文，参数个数，参数列表
  callback->Call(isolate->GetCurrentContext()->Global(), 1, argv);


}

void helloA(const FunctionCallbackInfo<Value>& args) {
  //...
}

void init(Handle<Object> exports) {
  NODE_SET_METHOD(exports, "hello", hello);
  NODE_SET_METHOD(exports, "helloA", helloA);
  //分别exports多个函数
}

NODE_MODULE(test, init);//指明导出模块名与初始化函数

```

更多的用法可以查看关于V8的c++开发接口


## 构建环境
新建``building.gyp``文件，用于``node-gyp``的构建
```json
{
 "targets": [
  {
   "target_name": "test",
   "sources": [ "test.cc" ]
  }
 ]
}
```
其中目标名指明等会渲染出来的目标文件，位置在``./build/Release/test.node``，可以直接引入使用。sources可以由多个文件组成。

在同文件下使用``node-gyp configure``配置环境

然后使用``node-gyp build``进行编译构建（如果cc文件没有出错的话不报错）

## 调用

在js中引入模块
```js
var test = require('./build/Release/test');
console.log(test.hello("intTest","23"));
console.log(test.helloA("intTest","23"));
```

## 业务实现

在cc文件与额外关联的h文件中进行所有的游戏的复杂计算，渲染成nodejs模块，由js引入。js自身完成网络服务的工作，也同时完成前台页面到后台数据计算的请求。
