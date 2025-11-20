### Ultrasound

 #### 接线图

![image-20251120203323567](C:\Users\xf\AppData\Roaming\Typora\typora-user-images\image-20251120203323567.png)

#### 实物图

![](C:\Users\xf\Pictures\Smart Car\接线图5-实物图-超声波.jpg)

#### code

```
#include "stm32f10x.h"
#include "Delay.h"

uint16_t Cnt = 0;
uint32_t timeout_flag = 0;

void Ultrasound_Init(void)
{
    // 开启时钟
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM4, ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB | RCC_APB2Periph_AFIO, ENABLE);
    
    // 配置Trig引脚（PB12）为推挽输出
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_12;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOB, &GPIO_InitStructure);
    
    // 配置Echo引脚（PB13）为上拉输入
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOB, &GPIO_InitStructure);
    
    // 初始化定时器4
    TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;
    TIM_TimeBaseInitStructure.TIM_Period = 60000 - 1;        // ARR: 最大计数值
    TIM_TimeBaseInitStructure.TIM_Prescaler = 72 - 1;        // PSC: 72分频，1MHz计数频率
    TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;
    TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;
    TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;
    TIM_TimeBaseInit(TIM4, &TIM_TimeBaseInitStructure);
    
    // 初始状态：Trig置低，定时器禁用
    GPIO_ResetBits(GPIOB, GPIO_Pin_12);
    TIM_Cmd(TIM4, DISABLE);
    TIM4->CNT = 0;
}


float Test_Distance(void)
{
    uint32_t timeout = 0;
    timeout_flag = 0;
    // 确保Trig初始为低电平
    GPIO_ResetBits(GPIOB, GPIO_Pin_12);
    Delay_us(2);
    // 发送Trig脉冲（10-20us高电平）
    GPIO_SetBits(GPIOB, GPIO_Pin_12);
    Delay_us(15);  // 15us脉冲
    GPIO_ResetBits(GPIOB, GPIO_Pin_12);
    // 等待Echo变为高电平（有超时保护）
    timeout = 0;
    while (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_13) == RESET) {
        timeout++;
        if (timeout > 100000) {  // 约100ms超时
            timeout_flag = 1;    // 标记超时
            return -1.0f;        // 返回错误值
        }
        Delay_us(1);
    }
    // 启动定时器并清零
    TIM4->CNT = 0;
    TIM_Cmd(TIM4, ENABLE);
    // 等待Echo变为低电平（有超时保护）
    timeout = 0;
    while (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_13) == SET) {
        timeout++;
        if (timeout > 60000) {   // 根据定时器ARR设置超时
            TIM_Cmd(TIM4, DISABLE);
            timeout_flag = 2;    // 标记超时
            return -2.0f;        // 返回错误值
        }
    }
    // 停止定时器并读取计数值
    TIM_Cmd(TIM4, DISABLE);
    Cnt = TIM_GetCounter(TIM4);
    // 计算距离（单位：厘米）
    // 计算公式：距离 = (时间 * 声速) / 2
    // 定时器频率1MHz，每计数1次=1us
    // 距离(cm) = (时间(us) × 340m/s × 100cm/m × 10^-6) / 2
    // 简化后：距离(cm) = (时间(us) × 0.034) / 2
    float distance = (Cnt * 0.034) / 2.0;
    // 清零定时器计数器
    TIM4->CNT = 0;
    // 防止测量过于频繁
    Delay_ms(60);
    return distance;
}
```

