虚继承例子1：
```c++
class A {};
class X: public virtual A {};
class Y: public virtual A {};
class Z: public X, public Y {};
```

64bit machine => A:1 byte; X: 8 byte; Y: 8 byte; Z: 16 byte
X和Y大小优化过的，只存了指向virtual base subobject的指针，8个字节
因此Z的大小应该是16字节

虚继承例子2：
```c++
class A { int a;};
class X: public virtual A {};
class Y: public virtual A {};
class Z: public X, public Y {};
```

64bit machine => A:4 byte; X: 16 byte; Y: 16 byte; Z: 24 byte
X和Y依然有指向virtual base subobject的指针，8字节，再加上4字节继承自A的成员int a， => 12
Z包含被X，Y，Z共享的唯一一个A的instance，4字节，加上X和Y分别去除instance的部分，8+8 = 16字节，最后alignment要求，补4个字节 => 24字节

总结：
1. empty base class“大概率（应该是标准行为了吧）”有编译器优化（derived class不继承空的一个字节）
2. derived class 只含有一份virtual base subobject实例