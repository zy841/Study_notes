# LCD

## 1. LCD相关知识介绍

### 1.1 LCD，OLED与Micro LED

**LCD:**

**根据液晶排列方式的不同**，可分为扭转式向列型（Twisted Nematic, TN）、超扭转式向列型（Super  Twisted Nematic, STN）、横向电场效应型（In Panel Switch，IPS）、垂直排列型 (Vertical Alignment，VA)。

**根据驱动形式的不同**，可分为无源矩阵液晶显示屏（Passive Matrix LCD，PMLCD），主要用于TN、 STN；有源矩阵液晶显示屏（Active Matrix LCD，AMLCD），主要用于TFT。

![image-20230410153250606](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101532689.png)



**OLED:**

OLED的工作原理比较简单，使用有机材料实现了类似半导体PN结 的功能效果，通电后有机发光二极管就发光，通的电越多，亮度越高，通过红、绿、蓝不同配比，实现组成各种颜色。

![image-20230410153518245](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101535293.png)



**LED显示屏和Micro LED显示屏：**

OLED使用有机发光二极管组成显示器，生活中常见的半导体发光二极管应用于户外广告牌等场景中（受结构、工艺、散热、成本等限制，无法做的很小，因此画质清晰度比较差，通常适合远距离观看）。

而Micro LED技术，将LED长度缩小到100μm以下，是常见LED的1%，比一粒沙子还要小。因为Micro LED单元过于微小，加大了制造的复杂性和更多的潜在问题，目前各企业正在积极研发中。

Micro LED没有LCD的液晶层，可以像OLED一样独立控制每个像素的开关和亮度，继承了几乎所有LCD和OLED的优点，如果后面Micro LED技术成熟，解决生产上的技术难题，那么Micro LED可能将会是下一代主流显示技术。

![image-20230410153935790](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101539826.png)



### 1.2 显示屏接口介绍

嵌入式图形系统组成示意图：

![image-20230410154531663](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101545704.png)

MCU根据代码内容计算需要显示的图像数据，然后将这些图像数据放入帧缓冲器。帧缓冲器本质是一块内存，因此也被称为GRAM（Graphic RAM）。帧缓冲器再将数据传给显示控制器，显示控制器将图像数据解析，控制显示屏对应显示。



帧缓冲器和显示控制器，可以集成在MCU内部，也可以和显示屏做在一起。对于大部分中、低端MCU， 不含显示控制器，内部SRAM也比较小，因此采用如下图所示的显示方案，将帧缓冲器、显示控制器和显示屏制作在一起，这样的屏幕习惯上称为“MCU屏”。

![image-20230410154739678](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101547720.png)



对于部分高端MCU或MPU，本身含有显示控制器，使用内部SRAM或外部SRAM，如图 38.1.10 所示， 通过并行的RGB信号和控制信号直接控制显示屏，这样的屏幕习惯上称为“RGB屏”。

![image-20230410154805982](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101548038.png)



MIPI（Mobile Industry Processor Interface，移动行业处理器接口）是ARM、ST、TI等公司成立的一个联盟，致力于定义和推广移动设备接口的规范标准化，从而减小移动设备的设计复杂度。



MIPI各接口总结：



![Snipaste_2023-04-10_16-05-07](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101605672.png)

对于STM32系列的MCU，不同型号对MIPI联盟显示接口的支持有所不同，总结如下：

-   所有STM32 MCU均支持MIPI-DBI C类接口（基于SPI协议）；
-   带FSMC的所有STM32 MCU均支持MIPI-DBI A类和B类接口；
-   带LTDC的STM32 MCU支持MIPI-DPI接口；
-   带DSI Host的STM32 MCU支持MIPI-DSI接口；

![image-20230410160707029](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101607080.png)



### 1.3 实验用显示屏控制器

STM32F103ZET6只有FSMC接口，没有LTDC或DSI Host，因此只能接MCU屏。

ILI9488是一款驱动IC，它可以驱动320*480分辨率，16.7M色彩深度的TFT  LCD，同时该芯片内部自带GRAM。

<img src="https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101626693.png" alt="image-20230410162620638"  />

①为控制引脚和信号引脚，支持DBI B类接口（Intel 8080接口）、DPI接口、MIPI DSI接口输入。经过索引寄存器（IR）、控制寄存器（CR）、 地址寄存器（AC）、读数据寄存器（RDR）、写数据寄存器（WDR）到GRAM。最后，GRAM把显示内容 传输到LCD屏幕的显示。

控制引脚IM[2:0]用于设置控制器的接口模式，如表 38.1.4 所示。显示屏支持的接口为16 Bit的MIPI-DBI  B类，因此IM[2:0]值为010，该值由屏幕供应商生产时硬件设置，此时用到的数据引脚为DB[15:0]。

![image-20230410163952200](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101639238.png)



Intel 8080接口16位连接示意图：

![image-20230410164114231](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101641262.png)

MCU和控制器之间，有4根控制信号，16个数据信号。CSX为片选信号，低电平有效（X表示低电平有效）；D/CX为数据/命令切换信号，低电平时DB传输的为命令，高电平时DB传输的为数据；WRX为写信号，低电平有效；RDX为读信号，低电平有效；DB[15:0] 为数据传输信号。



Intel 8080接口16位数据传输示意图：

![image-20230410164237690](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101642744.png)

首先片选信号CS拉低，复位信号RES保持为高。数据/命令切换信号D/C拉低，写信号引脚拉低，此时DB[15:0]发送的就是指令（黄色部分）；数据/命令切换信号D/C拉高，写信号引脚拉低，此时DB[15:0]发送的就是一个像素点的颜色数据（红、绿、蓝部分）， 其中低5位为蓝色数据、中6位为绿色数据、高5位为红色数据。



FSMC信号与8080信号对比

![image-20230410164659265](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101646297.png)

片选、读/写使能、数据总线都能对应，FSMC的地址总线和8080信号的数据/命令切换信号对应不上， 如何让两者对应上呢？假设NEx为NE4，即片选的Bank1的第四区，此时FSMC的寻址范围为0x6C000000~0x6FFF FFFF。**假设A23与D/C连接**，此时CPU访问(0x6C000000 + (1<<24) - 2))地址时，**A23为低电平， 对应D/C为命令信号**；当CPU访问(0x6C000000 + (1<<24))地址时，**A23为高电平，对应D/C为数据信号**。通过这个方式，即可实现地址总线与D/C的对应。



## 2.硬件设计

1.根据LCD屏幕设计的底板；2.开发板的LCD接口

![image-20230410165559768](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101655828.png)

①是连接LCD屏幕的FPC排座；；②是触摸芯片电路，用于实现触摸功能，放在后面一章单独讲解；③是背光驱动电路，通过控制三极管的通断，实现背光的开关，需要注意的是，控制引脚**支持PWM功能**，可以 实现亮度的调节；④是连接开发板的排针接口，该接口与开发板的LCD接口一致。

开发板LCD原理图详见开发板原理图介绍。

STM32使用FSMC接口实现LCD的Intel 8080总线接口，以及使用GPIO控制LCD其它功能，两者引脚的对应关系如下表所示：

![image-20230410170108098](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304101701145.png)



## 3.软件设计

软件设计思路：

1.   初始化定时器、PWM输出控制背光； 
2.   初始化FSMC接口； 
3.   初始化LCD控制器，LCD设置；
4.   实现LCD像素点显示、字符显示、数字显示、画线等功能； 
5.   主函数编写控制逻辑：清屏，使用LCD提供的函数，在指定位置显示字符、字符串、数字、图形等。

**步骤1**关键：

TIM3的计数器CNT从0开始计数到CCR，输出低电平，LCD背光亮，然后计数器CNT从CCR到ARR，输出高电平，LCD灭。即，CCR值越小，占空比越大，背光越暗，CCR值越大，占空比小，背光越亮，CCR值与亮度成正比。CCR的计数范围为0~2000，因此在TIM3中断回调函数里，修改CCR值，即可改变PWM占空比，实现背光不同亮度。



```C
/*** driver_timer.c 主要代码 */

//定时器初始化
TIM_HandleTypeDef htim;

void TimerInit(void)
{
    TIM_ClockConfigTypeDef sClockSourceConfig = {0};
    
    htim.Instance = TIM2;
    htim.Init.Prescaler = 72 - 1;
    htim.Init.CounterMode = TIM_COUNTERMODE_UP; //向上计数
    htim.Init.Period = 0;   //自动装载器ARR的值
    htim.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1; //时钟分频（与输入采样有关）
    //htim.Init.RepetitionCounter = 0;  //重复计数器值，仅存在于高级定时器
    htim.Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE; //不自动重新装载
    if(HAL_TIM_Base_Init(&htim) != HAL_OK)
    {
        Error_Handler();
    }
    sClockSourceConfig.ClockSource = TIM_CLOCKSOURCE_INTERNAL;  //内部时钟作为定时器时钟源
    if(HAL_TIM_ConfigClockSource(&htim, &sClockSourceConfig) != HAL_OK)
    {
        Error_Handler();
    }
}

void HAL_TIM_Base_MspInit(TIM_HandleTypeDef *htim)
{
    if(htim->Instance == TIM2)
    {
        __HAL_RCC_TIM2_CLK_ENABLE();
    }
}

//实现us级延时
void us_timer_delay(uint16_t t)
{
    uint16_t counter = 0;
    __HAL_TIM_SET_AUTORELOAD(&htim, t); //设置定时器自动加载值
    __HAL_TIM_SET_COUNTER(&htim, counter); //设置初始值
    HAL_TIM_Base_Start(&htim); //启动定时器
    while(counter != t)
    {
        counter = __HAL_TIM_GET_COUNTER(&htim); //获取定时器当前计数
    }
    HAL_TIM_Base_Stop(&htim);   //停止计时器
}

//PWM初始化
TIM_HandleTypeDef hpwm;
uint8_t lcd_brt;	//背光亮度
void TimerPWMInit(void)
{
    TIM_ClockConfigTypeDef sClockSourceConfig;
    TIM_OC_InitTypeDef sConfig;
    
    hpwm.Instance = TIMx;   //指定定时器3
    hpwm.Init.Prescaler = TIM_PRESCALER;    //预分频系数PSC = 360 - 1
    hpwm.Init.CounterMode = TIM_COUNTERMODE_UP;
    hpwm.Init.Period = TIM_PERIOD;  //ARR = 2000 - 1;
    //72MHz经过360分频后，定时器时钟为200KHz, 即计数器每隔5us计一次，从0计数到ARR，经历10ms
    hpwm.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;   //定时器时钟不从HCLK分频
    //hpwm.Init.RepetitionCounter = 0;
    hpwm.Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;

    if(HAL_TIM_PWM_Init(&hpwm) != HAL_OK)
    {
        Error_Handler();
    }

    sClockSourceConfig.ClockSource = TIM_CLOCKSOURCE_INTERNAL;
    HAL_TIM_ConfigClockSource(&hpwm, &sClockSourceConfig);
    //配置PWM的输入通道参数
    sConfig.OCMode = TIM_OCMODE_PWM1;   //PWM输出的两种模式：PWM1当极性为低，CCR < CNT, 输出低电平，反之高电平
    sConfig.OCPolarity = TIM_OCPOLARITY_LOW;
    sConfig.OCFastMode = TIM_OCFAST_DISABLE;    //输出比较快速使能禁止（仅在PWM1和PWM2可设置）
    sConfig.Pulse = LCD_BRT;    //在PWM1模式下，通道1（LCD_PWM）占空比
    if(HAL_TIM_PWM_ConfigChannel(&hpwm, &sConfig, TIM_LCD_PWM_CHANNEL)!= HAL_OK)
    {
        Error_Handler();
    }
}

void HAL_TIM_PWM_MspInit(TIM_HandleTypeDef *htim)
{
    GPIO_InitTypeDef GPIO_InitStruct;

    TIM_PWM_CLK_EN();           //PWM所涉及的TIM3时钟使能
    TIM_PWM_GPIO_CLK_EN();      //PWM所涉及的GPIOC时钟使能
    __HAL_RCC_AFIO_CLK_ENABLE(); //重映射涉及时钟使能
    //__HAL_AFIO_REMAP_TIM3_PARTIAL(); 
    //只有在全部重映射时，PC6才能重映射为TIM3_CH1,因此使用该函数使能
    __HAL_AFIO_REMAP_TIM3_ENABLE();//启动TIM3重映射

    GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;

    GPIO_InitStruct.Pin = TIM_LCD_PWM_PIN;
    HAL_GPIO_Init(TIM_LCD_PWM_PORT, &GPIO_InitStruct);

    HAL_NVIC_SetPriority(TIMx_IRQn, 0, 0);  //配置定时器中断优先级
    HAL_NVIC_EnableIRQ(TIMx_IRQn);          //使能TIM3中断
}

void TIM3_IRQHandler(void)
{
    HAL_TIM_IRQHandler(&hpwm);
}

void HAL_TIM_PWM_PulseFinishedCallback(TIM_HandleTypeDef *htim)
{
    if(htim->Instance == TIMx)
    {
        __HAL_TIM_SET_COMPARE(&hpwm, TIM_LCD_PWM_CHANNEL, lcd_brt *20);
    }
}
```

相关宏定义：

```C
/*driver_timer.h*/
#define TIMx                     TIM3
#define TIMx_IRQn                TIM3_IRQn
#define TIMx_IRQHandler          TIM3_IRQHandler
#define TIM_PRESCALER            ((36*10)-1)      
#define TIM_PERIOD               (2000-1)         
#define LCD_BRT                  (0)             

#define TIM_LCD_PWM_PIN          GPIO_PIN_6
#define TIM_LCD_PWM_PORT         GPIOC
#define TIM_LCD_PWM_CHANNEL      TIM_CHANNEL_1

#define TIM_PWM_CLK_EN()        __HAL_RCC_TIM3_CLK_ENABLE()
#define TIM_PWM_GPIO_CLK_EN()   __HAL_RCC_GPIOC_CLK_ENABLE()
```



**步骤2**

初始化FSMC，外接设备为Intel 8080接口的LCD，可以看作SRAM设备，可以使用模式1或模式A，在后面使能了扩展模式，因此这里只能使用模式A。

```C
/*协议部分初始化*/
static FSMC_NORSRAM_TimingTypeDef   hfsmc_rw;
static SRAM_HandleTypeDef           hsram;

void FSMC_LCD_Init(void)
{
    hfsmc_rw.AddressSetupTime = 0x00;
    //hfsmc_rw.AddressHoldTime = 0x01;
    hfsmc_rw.DataSetupTime = 0x02;
    //hfsmc_rw.BusTurnAroundDuration = 0x00;
    //hfsmc_rw.CLKDivision = 0x00;
    //hfsmc_rw.DataLatency = 0x00;
    hfsmc_rw.AccessMode = FSMC_ACCESS_MODE_A;

    hsram.Instance = FSMC_NORSRAM_DEVICE;
    hsram.Extended = FSMC_NORSRAM_EXTENDED_DEVICE;
    //硬件上使用FSMC_NE4连接的SRAM，因此对应BANK1(NORSRAM)的区域4；
    hsram.Init.NSBank = FSMC_NORSRAM_BANK4;
    hsram.Init.DataAddressMux = FSMC_DATA_ADDRESS_MUX_DISABLE;
    hsram.Init.MemoryType = FSMC_MEMORY_TYPE_SRAM;
    hsram.Init.MemoryDataWidth = FSMC_NORSRAM_MEM_BUS_WIDTH_16;
    //hsram.Init.BurstAccessMode = FSMC_BURST_ACCESS_MODE_DISABLE;
    //hsram.Init.WaitSignalPolarity = FSMC_WAIT_SIGNAL_POLARITY_LOW;
    //hsram.Init.WaitSignalActive = FSMC_WAIT_TIMING_BEFORE_WS;
    hsram.Init.WriteOperation = FSMC_WRITE_OPERATION_ENABLE;
    //hsram.Init.WaitSignal = FSMC_WAIT_SIGNAL_DISABLE;
    hsram.Init.ExtendedMode = FSMC_EXTENDED_MODE_ENABLE;    //使用扩展模式，使用模式A
    //hsram.Init.Asynchronous = FSMC_ASYNCHRONOUS_WAIT_DISABLE;
    //hsram.Init.WriteBurst = FSMC_WRITE_BURST_DISABLE;
    if(HAL_SRAM_Init(&hsram, &hfsmc_rw, &hfsmc_rw) != HAL_OK)
    {
        Error_Handler();
    }
}
/*硬件部分初始化*/
void HAL_SRAM_MspInit(SRAM_HandleTypeDef *hsram)
{
    GPIO_InitTypeDef    GPIO_InitStruct = {0};
 
    __HAL_RCC_FSMC_CLK_ENABLE();                     // 使能FSMC时钟
	__HAL_RCC_GPIOC_CLK_ENABLE();                    // 使能GPIO端口C时钟
	__HAL_RCC_GPIOD_CLK_ENABLE();                    // 使能GPIO端口D时钟
	__HAL_RCC_GPIOE_CLK_ENABLE();                    // 使能GPIO端口E时钟
	__HAL_RCC_GPIOF_CLK_ENABLE();                    // 使能GPIO端口F时钟
	__HAL_RCC_GPIOG_CLK_ENABLE();                    // 使能GPIO端口G时钟

    GPIO_InitStruct.Mode      = GPIO_MODE_OUTPUT_PP; // 配置为推挽输出功能
    GPIO_InitStruct.Pull      = GPIO_PULLUP;         // 上拉
    GPIO_InitStruct.Speed     = GPIO_SPEED_FREQ_HIGH;// 引脚翻转速率快
    
    GPIO_InitStruct.Pin = LCD_RST_PIN;
    HAL_GPIO_Init(GPIOF, &GPIO_InitStruct); //按上面参数配置LCD_RST引脚（PF11）

    GPIO_InitStruct.Pin = LCD_CS_PIN;
    HAL_GPIO_Init(GPIOG, &GPIO_InitStruct);
    
    GPIO_InitStruct.Mode      = GPIO_MODE_AF_PP;     // 配置为复用推挽功能
    GPIO_InitStruct.Pull      = GPIO_NOPULL;         // 不上拉
    GPIO_InitStruct.Speed     = GPIO_SPEED_FREQ_HIGH;// 引脚翻转速率快

    // PDx
    GPIO_InitStruct.Pin = LCD_GPIOD_PIN;
    HAL_GPIO_Init(GPIOD, &GPIO_InitStruct);      // 按上面参数配置D组涉及的GPIO
    
    // PEx
    GPIO_InitStruct.Pin = LCD_GPIOE_PIN;           
    HAL_GPIO_Init(GPIOE, &GPIO_InitStruct);      // 按上面参数配置E组涉及的GPIO
}
```



**步骤3.初始化LCD控制器**

使用初始化后的FSMC接口，访问LCD控制器ILI9488。

`driver_lcd.h`相关定义

```C
/*************** LCD延时函数重定义 ***************/
#define lcd_delay_ms(m)             HAL_Delay(m)
#define lcd_delay_us(u)             us_timer_delay(u)
/*************** LCD 常用颜色定义 ***************/
#define RGB888_TO_RGB565(color) ((((color) >> 19) & 0x1F) << 11) | \
                                ((((color) >> 10) & 0x3F) << 5) | \
                                (((color) >> 3) & 0x1F)
#define WHITE         	 (RGB888_TO_RGB565(0xFFFFFF))
#define BLACK         	 (RGB888_TO_RGB565(0x000000))	 
#define RED           	 (RGB888_TO_RGB565(0xFF0000))
#define GREEN         	 (RGB888_TO_RGB565(0x00FF00))
#define BLUE         	 (RGB888_TO_RGB565(0x0000FF)) 
#define YELLOW        	 (RGB888_TO_RGB565(0xFFFF00)) 

/*************** LCD 结构体定义 ***************/

/* LCD 指令结构体，用于读写LCD数据和寄存器 */
typedef struct
{
    __IO uint16_t reg;   // 寄存器值
    __IO uint16_t data;  // 数据值
}_lcd;   

/* LCD 设备信息 */
typedef struct
{
    uint16_t dev_id;    // 设备ID
    uint16_t hor_res;   // LCD水平显示长度
    uint16_t ver_res;   // LCD垂直显示长度
    uint16_t scan_mode; // 扫描模式
    
    uint16_t gram_cmd;  // 写入GRAM的指令
    uint16_t setx_cmd;  // 设置X坐标的指令
    uint16_t sety_cmd;  // 设置Y坐标的指令
}_lcddev;   

/* LCD 显示颜色结构体，用于设置背景颜色和笔迹颜色 */
typedef struct
{
    uint16_t backcolor; // 背景颜色值
    uint16_t textcolor; // 笔迹颜色值
}_lcd_color;    

/********** 全局变量声明 **********/
extern _lcddev    lcddev;
extern _lcd_color lcd_color;

/*************** 读写API声明 ***************/

/* 函数名：  void LCD_Write_Cmd(uint16_t _cmd)
 * 输入参数：_cmd->要写入的指令
 * 输出参数：无
 * 返回值：  无
 * 函数作用：往LCD写入指令
*/
extern void LCD_Write_Cmd(uint16_t _cmd);

/* 函数名：  void LCD_Write_Data(uint16_t _data)
 * 输入参数：_data->要写入的数据
 * 输出参数：无
 * 返回值：  无
 * 函数作用：往LCD写入数据
*/
extern void LCD_Write_Data(uint16_t _data);

/* 函数名：  uint16_t LCD_Read_Data(void)
 * 输入参数：无
 * 输出参数：无
 * 返回值：  返回寄存器的值
 * 函数作用：返回寄存器的值
*/
extern uint16_t LCD_Read_Data(void);

/*************** 颜色设置API声明 ***************/
extern void     LCD_SetColor(uint16_t _backcolor, uint16_t _textcolor);

/*************** LCD功能函数声明 ***************/
extern void     LCD_Scan_Dir(uint8_t _dir);
extern void     LCD_GRAM_Scan(uint8_t _opt);
extern void     LCD_SetCursor(uint16_t x, uint16_t y);
extern void     LCD_Clear(uint16_t _color);
extern void     LCD_Color_Fill(uint16_t x_start, uint16_t y_start, uint16_t x_end, uint16_t y_end, uint16_t color);

extern uint32_t LCD_Pow(uint8_t m, uint8_t n);
extern void     LCD_DrawPoint(uint16_t x, uint16_t y, uint16_t color);
extern void     LCD_ShowChar(uint16_t x, uint16_t  y, uint8_t num, uint8_t size, uint8_t mode);
extern void     LCD_ShowNum(uint16_t x,uint16_t y,uint32_t num, uint8_t size);
extern void     LCD_ShowString(uint16_t x, uint16_t y, uint8_t size, char *p);
extern void     LCD_DrawLine(uint16_t x1, uint16_t y1, uint16_t x2, uint16_t y2, uint16_t color);

/********** LCD初始化函数声明 **********/
extern void LCD_Init(void);
```



随后定义结构体“_lcd”的地址为“0x6D000000-2”，则成员“reg”地址为“0x6D000000-2”，成员“data” 的地址为“0x6D000000”，以后访问这两个地址，就可以分别发送命令、数据内容。

```C
/*driver_lcd.c中写命令与数据内容的函数*/
#define LCD		((__IO _lcd*)(0x6C000000 + (1 << 24) - 2))

void LCD_Write_Cmd(uint16_t _cmd)
{
    LCD->reg = _cmd;
}

void LCD_Write_Data(uint16_t _data)
{
    LCD->data = _data;
}
```



**LCD控制器初始化**

```C
static void ILI9488_RegConfig(void)
{
	LCD_Write_Cmd(0x3A);	//Interface Pixel Format
	LCD_Write_Data(0x05); 	//DBI-16bit
    
    LCD_Write_Cmd(0xB6); // Display Function Control
	LCD_Write_Data(0x02); // RM:System interface
	LCD_Write_Data(0x22); // SS:S960->S1 GS:G1->G480
	LCD_Write_Data(0x3B); // NL
    
    LCD_Write_Cmd(0x11); // Sleep OUT
	LCD_Write_Data(0x00); // No parameter 发送指令后，发送一个空数据占位
	LCD_Write_Cmd(0x29); // Display ON
	LCD_Write_Data(0x00); // No parameter 发送指令后，发送一个空数据占位
}
```



**LCD扫描相关函数**

```C
/*driver_lcd.c*/
//设置LCD显示方向
void LCD_Scan_Dir(uint8_t _dir)
{
    // bit[7] MY:Row Address Order
	// bit[6] MX:Column Address Order
	// bit[5] MV:Row/Column Exchange
	// bit[3] BGR:RGB-BGR Order
    _dir = (_dir << 5) | 0x08;	//获取高3位值， RGB to BGR
    LCD_Write_Cmd(0x36);		//Memory Access Control
    LCD_Write_Data(_dir);		//根据scan_mode的值设置LCD方向，共0~7种模式
}

//设置LCD的扫描方式
void LCD_GRAM_Scan(uint8_t _opt)
{
    uint8_t tmp = _opt % 2;
    if(_opt > 7)
        _opt = 0;
    
    lcddev.setx_cmd = 0x2A; // Column Address Set
	lcddev.sety_cmd = 0x2B; // Page Address Set
	lcddev.gram_cmd = 0x2C; // Memory Write
	lcddev.scan_mode = _opt; // 记录扫描模式以便之后判断屏幕方向
    
    if(tmp == 0) // 0 2 4 6 竖屏
	{
		if(lcddev.dev_id == 0x9488)
		{
			lcddev.hor_res = 320;
			lcddev.ver_res = 480;
		}
	}
	else if(tmp == 1) // 1 3 5 7 横屏
	{
		if(lcddev.dev_id == 0x9488)
		{
			lcddev.hor_res = 480;
			lcddev.ver_res = 320;
		}
	}
	LCD_Scan_Dir(lcddev.scan_mode);
}

uint16_t LCD_Read_Data(void)
{
	return LCD->data;
}

void LCD_Rest(void)
{
    LCD_RST(0);
    lcd_delay_us(0xAFF);
    LCD_RST(1);
    lcd_delay_us(0xAFF);
}

void LCD_GetDevID(void)
{
    uint16_t tmp = 0;
    LCD_Write_Cmd(0xD3);	//read ID4
    tmp = LCD_Read_Data();	//虚拟读取周期
    tmp = LCD_Read_Data();	//IC型号第一部分，值为x00
    tmp = tmp << 16;
    tmp = LCD_Read_Data();	//IC型号第二部分，值为x94
    tmp = tmp << 8;
    tmp |= LCD_Read_Data();//IC型号第三部分，值为x88
    
    lcddev.dev_id = tmp;
}

void LCD_Init(void)
{
    LCD_Backlight_Enable();
    LCD_Brightness(90);
    
    LCD_Rest();
    HAL_Delay(10);
    LCD_GetDevID();
    if(lcddev.dev_id == 0x9488)
    {
        ILI9488_Init();
    }
}
```

Memory Access Control寄存器的Bit[7:5]三位一起控制内存的读写方向。MY（Bit[7]）控制行地址顺序，MX（Bit[6]）控制列地址顺序，MV（Bit[5]）控制行列交换。

**显示效果**：

![image-20230411105235939](https://xiexun.oss-cn-hangzhou.aliyuncs.com/img2023/202304111052086.png)



**LCD显示功能函数**

```C
void LCD_SetColor(uint16_t _backcolor, uint16_t _textcolor)
{
    lcd_color.backcolor = _backcolor;
	lcd_color.textcolor = _textcolor;
}

void LCD_Backlight_Enable(void)
{
    if(HAL_TIM_PWM_Start_IT(&hpwm, TIM_LCD_PWM_CHANNEL) != HAL_OK)
    {
        Error_Handler();
    }
}

void LCD_Backlight_Disable(void)
{
    if(HAL_TIM_PWM_Stop_IT(&hpwm, TIM_LCD_PWM_CHANNEL) != HAL_OK)
    {
        Error_Handler();
    }
}

void LCD_Brightness(uint8_t value)
{
    if(value > 100)
        value = 100;
    lcd_brt = value;
}

void LCD_SetCursor(uint16_t x, uint16_t y)
{
    LCD_Write_Cmd(lcddev.setx_cmd);	//设置x坐标指令
    LCD_Write_Data(x >> 8);						//开始地址高8位
    LCD_Write_Data(x & 0xFF);					//开始地址低8位
    LCD_Write_Data((lcddev.hor_res - 1) >> 8);	//结束地址高8位
    LCD_Write_Data((lcddev.hor_res - 1) & 0xFF);//结束地址低8位
    
    LCD_Write_Cmd(lcddev.sety_cmd);	//设置y坐标指令
    LCD_Write_Data(y >> 8);						//开始地址高8位
    LCD_Write_Data(y & 0xFF);					//开始地址低8位
    LCD_Write_Data((lcddev.ver_res - 1) >> 8);	//结束地址高8位
    LCD_Write_Data((lcddev.ver_res - 1) & 0xFF);//结束地址低8位
}

void LCD_Clear(uint16_t _color)
{
    uint32_t i = 0;
    uint32_t totalpoint = lcddev.hor_res * lcddev.ver_res;
    
    lcd_color.backcolor = _color;
    LCD_SetCursor(0, 0);
    
    LCD_Write_Cmd(lcddev.gram_cmd);	//写入GRAM的指令
    for(i = 0; i < totalpoint; i ++)	//循环写像素颜色
    {
        LCD_Write_Data(lcd_color。backcolor);
    }
}

void LCD_Color_Fill(uint16_t x_start, uint16_t y_start, uint16_t x_end, uint16_t y_end, uint16_t color)
{
    uint16_t i, j;
    uint16_t xlen = 0;
    
    xlen = x_end - x_start + 1;
    for(i = y_start; i <= y_end; i ++)
    {
        LCD_SetCursor(x_start, i);
        LCD_Write_Cmd(lcddev.gram_cmd);
        for(j = 0; j < xlen; j ++)
        {
            LCD_Write_Data(color);
        }
    }
}

void LCD_DrawPoint(uint16_t x, uint16_t y, uint16_t color)
{
    LCD_Write_Cmd(lcddev.setx_cmd);
    LCD_Write_Data(x >> 8);
	LCD_Write_Data(x & 0xFF);
	LCD_Write_Data((lcddev.hor_res - 1) >> 8);
	LCD_Write_Data((lcddev.hor_res - 1) & 0xFF);

	LCD_Write_Cmd(lcddev.sety_cmd);
	LCD_Write_Data(y >> 8);
	LCD_Write_Data(y & 0xFF);
	LCD_Write_Data((lcddev.ver_res - 1) >> 8);
	LCD_Write_Data((lcddev.ver_res - 1) & 0xFF);

	LCD_Write_Cmd(lcddev.gram_cmd); 
	LCD_Write_Data(color);
}

void LCD_DramLine(uint16_t x1, uint16_t y1, uint16_t x2, uint16_t y2, uint16_t color)
{
    uint16_t t;
    int xerr = 0, yerr = 0, delta_x, delta_y, distance;
    int incx, incy, uRow, uCol;
    delta_x = x2 - x1;
    delta_y = y2 - y1;
    uRow = x1;
    uCol = y1;
    
    if(delta_x > 0) incx = 1;	//设置单步方向
	else if(delta_x == 0) incx = 0; //垂线
	else {incx = -1; delta_x = -delta_x;}
	if(delta_y > 0) incy = 1;	//设置单步方向
	else if(delta_y == 0) incy = 0; //水平线
	else {incy = -1; delta_y = -delta_y;}
    
    if(delta_x > delta_y) distance = delta_x;	//基本增量坐标轴
	else distance = delta_y;

	for(t = 0; t <= distance + 1; t++)
	{
		LCD_DrawPoint(uRow, uCol, color);
		xerr += delta_x;
		yerr += delta_y;
		if(xerr > distance)
		{
			xerr -= distance;
			uRow += incx;
		}
		if(yerr > distance)
		{
			yerr -= distance;
			uCol += incy;
		}
	}
}

void LCD_ShowChar(uint16_t x, uint16_t y, uint8_t num, uint8_t size, uint8_t mode)
{
    uint8_t data, t, t1;
    uint16_t y0 = y;
    //得到字体一个字符对应点阵集所占的字节数
    uint8_t csize(size / 8 + ((size % 8) ? 1 : 0)) * (size / 2);
    num = num - ' ';
    
    for(t = 0; t < csize; t ++)
    {
        if(size == 12)
			data = asc2_1206[num][t];	//调用1206字体
		else if(size == 16)
			data = asc2_1608[num][t];	//调用1608字体
		else if(size == 24)
			data = asc2_2412[num][t];	//调用2412字体
		else
			return;
        
        for(t1 = 0; t1 < 8; t1 ++)
        {
            if(data & 0x80)	//依次画最高位像素点
                LCD_DrawPoint(x, y, lcd_color.textcolor);
            else if(mode == 0)
                LCD_DrawPoint(x, y, lcd_color.backcolor);
            
            data <<= 1;
            y ++;
            if(y > lcddev.ver_res)	//超出垂直显示区域
                return;
            if((y - y0) == size)	//一列画完
            {
                y = y0;
                x ++;
                if(x > lcddev.hor_res)	//超出水平显示区域
                    return;
                break;
            }
        }
    }
}

void LCD_ShowString(uint16_t x, uint16_t y, uint8_t size, char *p)
{
    while((*p <= '~') && (*p >= ' '))
    {
        LCD_ShowChar(x, y, *p, size, 0);
        x += size / 2;	//字符位置水平偏移
		p ++;	//指向下一个字符
        if(x > (lcddev.hor_res - size / 2))	//超出水平显示区域
		{
			x = 0;
			y += size;
		}
        if(y > (lcddev.ver_res - size))
			break;
    }
}

uint32_t LCD_Pow(uint8_t m, uint8_t n)
{
    uint32_t result = 1;
	while(n --)	result *= m;
	return result;
}

void LCD_ShowNum(uint16_t x, uint16_t y, uint32_t num, uint8_t size)
{
    uint8_t len = 0;
    uint8_t t = 0, temp = 0, enshow = 0;
    uint32_t num_t = num;
    
    while(num_t)
    {
        len ++;
        num_t /= 10;
    }
    for(t = 0; t < len; t ++)
    {
        temp = (num / LCD_Pow(10, len - t - 1)) % 10;
        if(enshow == 0 && t < (len - 1))
        {
            if(temp == 0)
            {
                LCD_ShowChar(x + (size / 2) * t, y, ' ', size, 0);
                continue;
            }
            else
                enshow = 1;
        }
        LCD_ShowChar(x + (size / 2) * t, y, temp + '0', size, 0);
    }
}
```



**main函数控制逻辑**

```C
TimerInit(); // 定时器初始化
TimerPWMInit(); // PWM 初始化
FSMC_LCD_Init(); // FSMC 接口初始化

LCD_Init(); // LCD 初始化
LCD_Clear(WHITE); // 清屏为白色
LCD_ShowChar(0, 0, 'A', 24, 0); // 显示字符
LCD_SetColor(BLACK, WHITE); // 设置字符串颜色和背景色,显示字符串
LCD_ShowString(0, 24, 24, "www.100ask.net");
LCD_SetColor(WHITE, RED); // 设置字符串颜色和背景色,显示数字
LCD_ShowNum(0, 48, 123456789, 24);
LCD_DrawLine(160, 120, 70, 310, BLUE); // 画线显示三角形
LCD_DrawLine(70, 310, 250, 310, BLUE);
LCD_DrawLine(250, 310, 160, 120, BLUE);
LCD_Color_Fill(110,311, 210,411, GREEN); // 填充矩形
```

