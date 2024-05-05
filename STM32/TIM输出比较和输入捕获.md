# 输入捕获和输出比较的异同
或者说是容易混淆的地方
* 有四个捕获/比较寄存器，分别对应的四个通道。这些寄存器是输出/输入的模式下都公用的。
* 输出模式下：可读可写，只看寄存器和右半边
* 输入模式下：只读


还是从这张图开始：
![[Pasted image 20240417211122.png]]

# 输出比较
OC (Output Compare)。输出比较可以通过比较CNT 和CCR 寄存器之间的关系，来对输出电平进行置1，置0 或者是翻转的操作，用于输出有一定频率和占空比的PWM 波形，CCR 的值是通过我们自己设置的，CNT 是自增的。
## PWM 波形
**_PWM（Pulse Width Modulation）脉冲宽度调制_** 在具有惯性的系统中，可以通过对一系列脉冲的宽度进行调制，来等效地获得所需要的模拟参量，常应用于电机控速等领域。也就是用来**等效地实现一个模拟信号**的输出（左图虚线余弦），也就是最开始提出的问题：数字输出端口控制LED，按理说只有完全亮和完全灭两种状态，但通过PWM就可实现呼吸灯，当不断点亮熄灭点亮熄灭...的频率足够大时就不会时闪烁而是呈现中等亮度！！
![[Pasted image 20240503141804.png]]
* 占空比Duty = CCR / (ARR + 1)
* PWM频率：  Freq = CK_PSC / (PSC + 1) / (ARR + 1)
* PWM分辨率：  Reso = 1 / (ARR + 1)
![[Pasted image 20240503142000.png]]
输出比较模式：

|    模式    |                                            描述                                            |     |
| :------: | :--------------------------------------------------------------------------------------: | --- |
|    冻结    |                                  CNT = CCR 时，REF 保持为原状态                                  |     |
| 匹配时置有效电平 |                                   CNT = CCR，REF 置有效电平                                    |     |
| 匹配时置无效电平 |                                   CNT = CCR，REF 置无效电平                                    |     |
| 匹配时电平翻转  |                                    CNT = CCR，REF电平翻转                                     |     |
| 强制为无效电平  |                                   CNT与CCR无效，REF强制为无效电平                                   |     |
| 强制为有效电平  |                                   CNT与CCR无效，REF强制为有效电平                                   |     |
|  PWM模式1  | 向上计数：CNT<CCR时，REF置有效电平，CNT≥CCR时，REF置无效电平<br><br>向下计数：CNT>CCR时，REF置无效电平，CNT≤CCR时，REF置有效电平 |     |
|  PWM模式2  | 向上计数：CNT<CCR时，REF置无效电平，CNT≥CCR时，REF置有效电平<br><br>向下计数：CNT>CCR时，REF置有效电平，CNT≤CCR时，REF置无效电平 |     |

```c
void PWM_Init(void){
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE); // 开启定时器时钟

    // 配置GPIO
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE); // 开启GPIOA时钟
    GPIO_InitTypeDef GPIO_InitStruct;
    GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP; // 复用推挽输出
    GPIO_InitStruct.GPIO_Pin = GPIO_Pin_0; // PA0
    GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_InitStruct);

    // 使用内部时钟
    TIM_InternalClockConfig(TIM2);
    // 配置时基单元
    TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
    TIM_TimeBaseInitStruct.TIM_ClockDivision = TIM_CKD_DIV1; // 不分频 用于滤波
    TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
    TIM_TimeBaseInitStruct.TIM_Period = 100- 1;  // ARR
    TIM_TimeBaseInitStruct.TIM_Prescaler = 720 - 1; // PSC
    TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
    TIM_TimeBaseInit(TIM2, &TIM_TimeBaseInitStruct);
    
    // 配置输出比较单元
    TIM_OCInitTypeDef TIM_OCInitStruct;
    TIM_OCStructInit(&TIM_OCInitStruct); // 将结构体初始化为默认值
    TIM_OCInitStruct.TIM_OCMode = TIM_OCMode_PWM1; // PWM模式1
    TIM_OCInitStruct.TIM_OCPolarity = TIM_OCPolarity_High; // 极性就是高电平有效。如果是低电平有效，就是TIM_OCPolarity_Low
    TIM_OCInitStruct.TIM_OutputState = TIM_OutputState_Enable; 
    TIM_OCInitStruct.TIM_Pulse = 0; // 设置CCR寄存器的值
    TIM_OC1Init(TIM2, &TIM_OCInitStruct);

    TIM_Cmd(TIM2, ENABLE); // 使能TIM2
}
```
# 输出比较
* IC (Input Capture)
* 输入捕获模式下，**当通道输入引脚出现指定电平跳变时**，**当前CNT的值将被锁存到CCR中**，可用于测量PWM波形的频率、占空比、脉冲间隔、电平持续时间等参数