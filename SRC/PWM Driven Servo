## Smart Car

### 材料

1. 小车底盘
2. stm32f106c8t6
3. 电机驱动模块tb6612
4. 蓝牙模块hc06
5. HC-SR04超声波测距模块  3.3v-5v
6. TCRT5000红外寻轨迹模块  4个
7. sg90舵机 一个
8. 电源转换模块

#### 材料到了，进行简单拼接，电机进行焊接

![](C:\Users\xf\Pictures\Smart Car\拼接.jpg)

___

### PWM Driven Servo

#### 一个tb6612最大只能驱动两个电机，我用串联方式驱动了4个电机

#### 接线图

![image-20251116233746110](C:\Users\xf\AppData\Roaming\Typora\typora-user-images\image-20251116233746110.png)



#### 实物图

![](C:\Users\xf\Pictures\Smart Car\电机.jpg)



### code

```
PWM驱动电机
#include "stm32f10x.h"

void PWM_Init(void)
{
    // 1. 使能时钟
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    // 2. 配置GPIO为复用推挽输出
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;  // 复用推挽
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1; // PA0(通道1), PA1(通道2)
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    // 3. 配置时基单元
    TIM_InternalClockConfig(TIM2);  // 使用内部时钟
    
    TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;
    TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;
    TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;
    TIM_TimeBaseInitStructure.TIM_Period = 100 - 1;  // ARR: PWM周期 = (ARR+1)/TIMx_CLK
    TIM_TimeBaseInitStructure.TIM_Prescaler = 36 - 1; // PSC: 72MHz/(36)=2MHz
    TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;
    TIM_TimeBaseInit(TIM2, &TIM_TimeBaseInitStructure);
    
    // 4. 配置两个PWM通道（通道1和通道2）
    TIM_OCInitTypeDef TIM_OCInitStructure;
    
    // 通道1配置（PA0）
    TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;
    TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;
    TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;
    TIM_OCInitStructure.TIM_Pulse = 0;  // 初始占空比=0
    TIM_OC1Init(TIM2, &TIM_OCInitStructure);  // 初始化通道1
    
    // 通道2配置（PA1）
    TIM_OCInitStructure.TIM_Pulse = 0;
    TIM_OC2Init(TIM2, &TIM_OCInitStructure);  // 初始化通道2
		
    // 5. 启动定时器
    TIM_Cmd(TIM2, ENABLE);
}

// 设置通道1占空比（PA0） 右边
void PWM_SetCompare1(uint16_t Compare)
{
    TIM_SetCompare1(TIM2, Compare);
}

// 设置通道2占空比（PA1） 左边
void PWM_SetCompare2(uint16_t Compare)
{
    TIM_SetCompare2(TIM2, Compare);
}

-------------------------------------
控制前后左右方向
#include "stm32f10x.h"                  // Device header
#include "PWM.h"

void Motor_Init(void)
{
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_4 | GPIO_Pin_5 | GPIO_Pin_6 | GPIO_Pin_7;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    GPIO_ResetBits(GPIOA, GPIO_Pin_4 | GPIO_Pin_5 | GPIO_Pin_6 | GPIO_Pin_7);
    
    PWM_Init();
}

void Motor_SetLeftSpeed(int8_t Speed)
{
	if (Speed >0)
	{
		GPIO_SetBits(GPIOA, GPIO_Pin_4);
		GPIO_ResetBits(GPIOA, GPIO_Pin_5);
		PWM_SetCompare2(Speed);
	}
	else if(Speed==0){
		GPIO_SetBits(GPIOA, GPIO_Pin_4);
		GPIO_SetBits(GPIOA, GPIO_Pin_5);
		PWM_SetCompare2(Speed);
	}else{
		GPIO_ResetBits(GPIOA, GPIO_Pin_4);
		GPIO_SetBits(GPIOA, GPIO_Pin_5);
		PWM_SetCompare2(-Speed);
	}
}
void Motor_SetRightSpeed(int8_t Speed)
{
	if (Speed >0)
	{
		GPIO_SetBits(GPIOA, GPIO_Pin_6);
		GPIO_ResetBits(GPIOA, GPIO_Pin_7);
		PWM_SetCompare1(Speed);
	}
	else if(Speed==0){
		GPIO_SetBits(GPIOA, GPIO_Pin_6);
		GPIO_SetBits(GPIOA, GPIO_Pin_7);
		PWM_SetCompare1(Speed);
	}else{
		GPIO_ResetBits(GPIOA, GPIO_Pin_6);
		GPIO_SetBits(GPIOA, GPIO_Pin_7);
		PWM_SetCompare1(-Speed);
	}
}


-------------------------------------
执行行动命令
#include "stm32f10x.h"                  // Device header
#include "Motor.h"
#include "Delay.h"
#include "OLED.h"

void Car_Init(){
    Motor_Init();
    OLED_Init();
}

void Go_Ahead(){
    OLED_Clear();
    OLED_ShowString(1, 1, "Going Ahead");
    Motor_SetLeftSpeed(99);
    Motor_SetRightSpeed(99);
}

void Go_Back(){
    OLED_Clear();
    OLED_ShowString(1, 1, "Going Back");
    Motor_SetLeftSpeed(-99);
    Motor_SetRightSpeed(-99);
}

void Turn_Left(){  
    OLED_Clear();
    OLED_ShowString(1, 1, "Turning Left");
    Motor_SetLeftSpeed(1);    
    Motor_SetRightSpeed(99);   
}

void Turn_Right(){
    OLED_Clear();
    OLED_ShowString(1, 1, "Turning Right");
    Motor_SetRightSpeed(1);   
    Motor_SetLeftSpeed(99);    
}

void Self_Left(){  
    OLED_Clear();
    OLED_ShowString(1, 1, "Self Left");
    Motor_SetLeftSpeed(-99);  
    Motor_SetRightSpeed(99);   
}

void Self_Right(){ 
    OLED_Clear();
    OLED_ShowString(1, 1, "Self Right");
    Motor_SetLeftSpeed(99);   
    Motor_SetRightSpeed(-99); 
}

void Car_Stop(){
    OLED_Clear();
    OLED_ShowString(1, 1, "Stopped");
    Motor_SetLeftSpeed(0);
    Motor_SetRightSpeed(0);
}
```




