## 基本概念

<br/>

- 将模拟信号转换为数字信号

- 工作方式
  - 并联比较型
    
    <img src="pic/41d1a1c0bade1db2243442cf0e2cddc5.png" alt="43d82e7d632cb131cfddc5b2a60a619.png" style="zoom:30%;" />
  - **逐次逼近型**
    
    <img src="pic/d4a94a21400378dab271e082982e1d31.png" alt="截图" style="zoom:30%;" />
- 主要参数
  - 精度：8bit、10bit、12bit、14bit、16bit
  - 分辨率（量化位数）：3.3V ，12位，4096份，0.805mV
  - 16个外部通道，3个内部通道
  - 转换方式：4个注入转换通道、16个规则转换通道

<img src="pic/2556831cc3841fbe034697b3fe2bce34.png" alt="截图" style="zoom:50%;" />

<br/>

<br/>

## 框图

<br/>

### 0x01、ADC框图

![截图](pic/23910a4cfbd088a91614bcb5331fb7b9.png)

<br/>

![截图](pic/4397c61bddb027c771c25703f0fe053b.png)

<br/>

<br/>

## 功能

### 0x01、ADC_CR1分辨率

![截图](pic/084f4d751d423088b8580c8edfe1c49b.png)![截图](pic/a87c60fe82f317ee5181ab3677ff08e8.png)

<br/>

### 0x02、时钟

![截图](pic/900725876165db6bc04b227a1752d55e.png)

![截图](pic/7762a768c1bda3719b604ef15ca6170e.png)

### 0x03、ADC_SQR1通道选择

![截图](pic/631cf21b965e5c5ed3cc77e9eaa853dd.png)![截图](pic/97f09da805446dcd74d2443847b75f6c.png)

<br/>

**配置步骤**

![截图](pic/9ca4f71278008869c021c42992f1b1d8.png)

![截图](pic/b421128c911666ff4d5f36c97e6d0149.png)

图例是规则通道的选择寄存器，注入通道的寄存器配置方法同上。

<br/>

### 0x04、ADC_CR2连续转换控制

![截图](pic/66a24248814b1ce266f4dce373bc2fa8.png)![截图](pic/7885594a91d171f5e4333f61d04bebc3.png)

使用`单次转换`可控性更好，使用SWSTART开始转换一次。

<br/>

### 0x05、ADC_CR1扫描模式

多通道排队转换时开启扫描模式，`单次转换`需要配合这个

![截图](pic/d19c39d68db04ce85cbf61f6c3b24f7b.png)

<br/>

### 0x06、ADC_CR2转换结束

**转换结束标志**

![截图](pic/bad1a892497810e4e305700554cc2ff8.png)![截图](pic/b889a6e57a277a776c2007518927b362.png)

<br/>

**转换结束配置位**

因为规则转换寄存器只有一个，因此当有多个排队时想要获得每个转换的数据就需要配置为1（或者使用DMA就无需配1），单个转换无需排队时就配置为0

多通道排队时：1

单通道时：0

![截图](pic/7956f4d270ff5038e1b6d708bde567da.png)![截图](pic/31132d9f8182d8832be9d52fa2bb94fb.png)

<br/>

### 0x07、ADC_CR2对齐

一般选择右对齐

![截图](pic/ede8e9de940e2c7b15427e967e1155e7.png)![截图](pic/54dbb30908f804a28a27f1785650735a.png)

### 0x08、ADC_SMPR采样时间

对应18个可配置的ADC；SMP4对应通道4（PA4）

![截图](pic/683b0ce5a95cb989f895f1eeb7ad10f6.png)![截图](pic/99125cc3164294b5ec48e4297737cb3b.png)

<br/>

### 0x09、温度传感

配置通道

![截图](pic/7a1c62197dc6ddcf27385675eb5d2993.png)

<br/>

<br/>

<br/>

<br/>

<br/>

## 示例代码

<br/>

```c_cpp
void ADC_Init(void)
{
    //打开时钟；GPIOA、ADC1
    RCC->AHB1ENR |= (0X1<<0);
    RCC->APB2ENR |= (0x1<<8);
    
    //配置GPIOA模拟输入
    GPIOA->MODER |= (0x3<<8);

    //配置ADC功能
    ADC1->CR1 &= ~(0x3<<24); //分辨率12位
    ADC1->CR1 &= ~(0x1<<8); //扫描模式
    ADC1->CR2 &= ~(0x1<<11);//数据右对齐
    ADC1->CR2 &= ~(0x1<<1);//单次转换
    ADC1->SMPR2 &= 0x7<<12;//通道4采样时间480个周期
    
    ADC1->SQR1 &= ~(0xf<<20);  //规则序列通道
    ADC1->SQR3 |= (0x4<<0);    //PA4

    ADC->CCR = ADC->CCR & ~(0x3<<16) | (1<<16); //4分频
    //开启ADC1
    ADC1->CR2 |= (0x1<<0);

}

u32 GET_ADC_Proportion(void)
{
    // 开始转换
    ADC1->CR2 |= (0x1 << 30);

    while (!(ADC1->SR & 0x1 << 1));

    u32 data = ADC1->DR;
    data = 100 - (data / 4096.0 * 100); //注意浮点数自动转换
    return data;
}
```