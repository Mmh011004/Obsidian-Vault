# TIM 定时中断
定时器类型

| **类型** |         **编号**          | **总线** |                         功能                         |
| :----: | :---------------------: | :----: | :------------------------------------------------: |
| 高级定时器  |       TIM1, TIM8        |  APB2  |      拥有通用定时器全部功能，并额外具有重复计数器、死区生成、互补输出、刹车输入等功能      |
| 通用定时器  | TIM 2, TIM3, TIM4, TIM5 |  APB1  | 拥有基本定时器全部功能，并额外具有内外时钟源选择、输入捕获、输出比较、编码器接口、主从触发模式等功能 |
| 基本定时器  |       TIM6, TIM7        |  APB1  |                 拥有定时中断、主模式触发DAC的功能                 |
下图为**时钟树**：
![[Pasted image 20240417200026.png]]
## 基本定时器
![[Pasted image 20240416224632.png]]
基本定时器的时钟源只能是来自内部时钟，其中内部时钟是 72MHz。但是从时钟树的图可以看出SYSCLK = 72MHz，AHB = 72MHz，APB1 = 36MHz，但是APB1 的分频系数是 2，所以会将 36 * 2 还是得到72MHz。
基本定时器主要涉及的三个寄存器分别是计数器寄存器(TIMx_CNT)，预分频器寄存器 (TIMx_PSC)，自动重载寄存器 (TIMx_ARR)，这三个寄存器都是 **16**位有效数字，可以设置值为 0~65535。
### 计数器
计数器计数频率：
$$\mathrm{CK\_CNT=CK\_PSC/(PSC+1)}$$ 其中,CK_PSC 表示预分频器的时钟频率，PSC 为预分频值，如果 PSC=0，则不进行分频，PSC=1 则是二分频，以此类推。预分频值和实际的预分频系数相差 1，所以需要相加 1。后面的**计数器**每经过一个上升沿就进行+1 操作，当与**自动重载寄存器**的值相等时，产生中断信号，并且将计数器清零。
### 自动重载器
自动重载寄存器中存放的就是目标值，当计数器的值达到目标值时，生成中断信号。
## 通用定时器
![[Pasted image 20240417211122.png]]
## 定时中断基本结构
![[Pasted image 20240417211201.png]]
从图中可以看出在时基单元的时钟选择部分，也就是黄色之前的部分，一共有六种选择的方式。
``` c
void TIM_InternalClockConfig(TIM_TypeDef* TIMx); // 选择内部时钟

void TIM_ETRClockMode1Config(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler, uint16_t TIM_ExtTRGPolarity, uint16_t ExtTRGFilter); // 选择通过ETR外部时钟模式1进行

void TIM_ETRClockMode2Config(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler, uint16_t TIM_ExtTRGPolarity, uint16_t ExtTRGFilter); // 选择通过ETR外部时钟模式2进行

void TIM_ITRxExternalClockConfig(TIM_TypeDef* TIMx, uint16_t TIM_InputTriggerSource); // 选择ITRx其他定时器

void TIM_TIxExternalClockConfig(TIM_TypeDef* TIMx, uint16_t TIM_TIxExternalCLKSource, uint16_t TIM_ICPolarity, uint16_t ICFilter); // 选择TIx捕获通道

void TIM_ETRConfig(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler, uint16_t TIM_ExtTRGPolarity, uint16_t ExtTRGFilter); // 这个是单独用来配置ETR引脚的预分频器、极性、滤波器这些参数
```
时基单元是通过结构体进行配置的
```c
typedef struct
 {
		uint16_t TIM_Prescaler;              // 预分频器
		uint16_t TIM_CounterMode;        // 计数模式
		uint32_t TIM_Period;                   // 定时器周期
		uint16_t TIM_ClockDivision;       // 时钟分频
		uint8_t TIM_RepetitionCounter; // 重复计算器
} TIM_TimeBaseInitTypeDef;
```
* **TIM_Prescaler**：定时器预分频器设置，时钟源经该预分频器才是定时器时钟，它设定TIMx_PSC 寄存器的值。可设置范围为 0 至 65535，实现 1至 65536 分频。为啥要搞一个预分频器，那是因为系统时钟频率太快了，90MHZ啊，这一般人定时器可顶不住这么快的速度，所以分频一下，让他的给定时器的时钟频率少一点，仅此而已。
* **TIM_CounterMode**：定时器计数方式，可是在为向上计数 **TIM_CounterMode_Up**、向下计数**TIM_CounterMode_Down**以及三种中心对齐**TIM_CounterMode_CenterAligned 1/2/3**模式。基本定时器只能是向上计数，即 TIMx_CNT只能从 0开始递增，并且无需初始化。
* **TIM_Period**：定时器周期，实际就是设定自动重载寄存器的值，在事件生成时更新到影子寄存器。可设置范围为 0至 65535。通过-1是由于计数是从 0 开始的。
* **TIM_ClockDivision**：时钟分频，设置定时器时钟 CK_INT 频率与数字滤波器采样时钟频率分频比，基本定时器没有此功能，不用设置。在一个固定的时钟频率f 进行采样，如果N 个采样点都是相同的电平，就代表输入信号稳定了。这样就能保证输出信号在一定程度上的滤波。三个值 **TIM_CKD_DIV1，TIM_CKD_DIV2，TIM_CKD_DIV4**，这分别是不分平、二分频和四分平
* **TIM_RepetitionCounter**：重复计数器，属于高级控制寄存器专用寄存器位，利用它可以非常容易控制输出 PWM 的个数。这里不用设置.
## 程序编程步骤
1. 开启总线时钟
2. 选择时基单元时钟 (6 种方式)
3. 配置时基单元 (上面的 TIM_TimeBaseInitTypeDef 结构体)
4. 使能中断 void TIM_ITConfig (TIM_TypeDef* TIMx, uint 16_t TIM_IT, FunctionalState NewState)，这样能够开启更新终端到NVIC 的通路。
5. 配置NVIC
## 更新事件和更新中断
