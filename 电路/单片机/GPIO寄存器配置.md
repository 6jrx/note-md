![截图](pic/e9c2b05c63bf9be57878e1690a67971d.png)

## 寄存器结构

<br/>

#### 0x01、GPIOx_MODER 端口模式

配置`输入`或`输出`

![截图](pic/4d3ce8e5076b9b82ee47d0fb5a73fea1.png)

```c_cpp
GPIOA->MODER = GPIOA->MODER & ~(0xf<<18) | 0xa<<18; // GPIOA9复用模式
```

<br/>

#### 0x02、GPIOx_OTYPER 端口输出类型

<img src="pic/4bf8e12b7bc35015938fe678e894f97b.png" alt="截图" style="zoom:50%;" />

<img src="pic/749e3036c65c1ecbe1e44ba157374e45.png" alt="截图" style="zoom:50%;" />

```c_cpp
GPIOA->OTYPER &= ~(0x3<<9);            // 推挽输出，同时配置两个IO口
```

<br/>

#### 0x03、GPIOx_OSPEEDR 端口输出速度

<img src="pic/429b4217f569b3d5a07afb1800f38cf2.png" alt="截图" style="zoom:50%;" />

```c_cpp
GPIOA->OSPEEDR = GPIOA->OSPEEDR & ~(0xf<<18) | 0xa<<18;  //配置为 高速，同时配置了两个口
```

<br/>

#### 0x04、GPIOx_PUPDR 端口上下拉

<img src="pic/93327160f6fb60ddcba266ee46f73941.png" alt="截图" style="zoom:50%;" />

<img src="pic/ec5686edbf2757167a96c1e649307129.png" alt="截图" style="zoom:50%;" />

```c_cpp
GPIOC->PUPDR &= ~(0xff << (2 * 4));     //同时配置4个口为无上下拉
```

<br/>

#### 0x05、GPIOx_IDR 输入数据

只在端口模式为`输入`模式时使用

<img src="pic/a475ae634af80dad6213e9116a0e4f87.png" alt="截图" style="zoom:50%;" />

```c_cpp
#define FIRE_War !(GPIOA->IDR & GPIO_IDR_IDR_4)   //通常用宏定义的方式来读取值，这里用到了内核库函数里的宏定义作偏移位
```

<br/>

#### 0x06、GPIOx_ODR 输出数据

<img src="pic/b9550ca85f7c4d20eb41388a8930956b.png" alt="截图" style="zoom:50%;" />

```c_cpp
#define LED4_FZ     GPIOC->ODR ^= (1 << 7)     //这里也使用宏定义来操作输出高低电平，异或，用来反转该位
```

<br/>

#### 0x07、GPIOx_AFRL 复用功能低位寄存器

低位可以复用的外设如下，4位控制一个

![截图](pic/1119c23f24a0fcfc0c800e25ddec84f6.png)

```c_cpp
GPIOA->AFR[0] = GPIOA->AFR[0] & ~(0xff<<4) | 0x77<<4;    //低位复用USART1
```

**复用地址属性是一个数组，引用时注意！！！**

<img src="pic/5090b81b6d12baa6b650b51f61d5b1dd.png" alt="截图" style="zoom:30%;" />

**高低位复用的外设选择！！！**

![截图](pic/5e4c871c4060a105e6094f06068db79c.png)

<br/>

#### 0x08、FPIOx_AFRH 复用功能高位寄存器

![截图](pic/b8f1bac1b78996327ffa2845d8a76c46.png)

```c_cpp
GPIOA->AFR[1] = GPIOA->AFR[1] & ~(0xff<<4) | 0x77<<4;    //高位复用USART1
```

或者查询芯片数据手册

![截图](pic/2f45ae46c2cd20fe965ec9ce5a8afda7.png)
