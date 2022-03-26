---
title: 怎么在c语言中调用c++的类非静态成员函数
date: 2022-01-06 21:58:35
tags:
  - c
  - c++
---

考虑到下面的代码。

我们有一个c库代码`lib.c`,里面有个函数叫`subscribe`,它接受一个callback函数指针作为参数。

```c
// lib.c

typedef int(*T)(int a);
void subscribe(void* callback) {
  T cb = reinterpret_cast<T>(callback);
  cb(20);
}
```

同时在我们的c++项目中，我想去调用这个`subscribe`函数，并准备传入一个非静态成员函数作为参数。

```c++
class Demo {
private:
    int base = 100;
public:
     int received(int offset);
};

int Demo::received(int offset) {
    printf("received data is %d\n", this->base + offset);
    return this->base + offset;
}

int main() {
    Demo* demo = new Demo();
    subscribe(reinterpret_cast<void*>(&Demo::received));// compiler error; reinterpret_cast from 'int (Demo::*)(int)' to 'void *' is not allowed
}
```

不幸的是，我们得到了一个编译错误，看起来像是个类型错误。

## 为什么会触发这个错误？

因为这个c++的非静态类成员函数有一个隐式的参数叫*this*.它是一个指针指向了这个类的实例对象本身。

```c++
Demo* demo = new Demo();
demo->received(20);
```

上面这行代码等价于下面的这行

```c++
Demo* demo = new Demo();
received(demo, 20); // recevicde(this, 20)
```

因此我们不能在c代码直接调用这个非静态类成员函数。但是类的静态成员函数没有这个约束。因为类的静态类成员函数并不和任意一个类实例对象绑定。

于是我们可以写出下面这段代码👇

```c++
class Demo {
private:
    int base = 100;
public:
  	int received(int offset) {
         printf("received data is %d\n", this->base + offset);
         return this->base + offset;
    }
  	static int static_received(int offset) {
         printf("static_received data is %d\n", offset);
    }
};

int main() {
    Demo* demo = new Demo();
    subscribe(reinterpret_cast<void*>(&Demo::static_received));// 👌
}
```

基于上面的原理,我们可以在静态成员函数中去调用这个类的非静态成员函数。也就意味着我们可以在c代码中间接的去调用非静态成员函数。

![call_tree](https://chromer-blog.oss-cn-shanghai.aliyuncs.com/blog/call_tree-ddeb964ee92a28827b5e886cfb26a97db4b318657ba37a5d71a5e3b81e0ee228.png)

但是要向做的这点，我们首先要解决的是如何在静态成员函数中去调用非静态成员。我们需要确保在类的静态成员中能访问到`this`指针，这样我们才能调用非静态成员函数。

但是上面说过静态成员函数不和任意一个类实例对象绑定。也就是说我们无法在类静态成员函数中直接获取到`this`指针，这是不是意味着这个方案死路一条？别急，我们从c++语言层面确实无法解决，但是我们可以从汇编代码入手。要确保在类静态成员函数中获取到`this`指针，我们需要在调用函数之前先把`this`指针保存到寄存器。然后我们再函数内部使用内联汇编的方式再从这个寄存器拿到`this`指针。整个过程的伪代码如下:

```c++
static int static_received(int data) {
  void* this_obj = nullptr;

  obj = get_this_obj_from_register();

  return static_cast<Demo*>(this_obj)->say(data);
}
```

