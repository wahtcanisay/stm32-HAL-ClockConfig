# stm32-HAL-ClockConfig
更改stm32单片机时钟频率实现板载LED闪烁时间变化  
## 硬件原理  
AHB分频器之后输出HCLK频率，连接到Cortex-M3内核，决定了程序运行的速度，可以通过板载LED闪烁快慢间接观察代码执行速度。  

## 软件实现

### cubemx部分
首先设置调试接口，选中`SYS`, `Debug`选中`Serial Wire`。  
点击右侧的`Clock Configuration`,这里显示的是时钟树。  
默认状态下，HCLK的输出是8MHz，从HSI固定的8MHz处获得。  
  
  
首先对PC13设置为输入，参数为初始高电平，开漏模式，低速。  

为了使用HCLK所允许的最大时频率，做出如下设置：  
左侧点击`RCC`模块，选择高速外部时钟HSE`Crystal/Ceramic Resonator`，晶体/陶瓷振荡器。此时可以看见右侧的PD0，PD1被设置为晶振的输入引脚和输出引脚。  
再在`Clock Configuration`中找到HSE的输入频率，设置为8  
锁相环PLL中选择HSE作为输入，倍频系数`*PLLMul`设置为9  
最后选择`PLLCLK`作为`SYSLCK`的输入,AHB分频系数设置为`/1`，APB1分频器设置为`/2`，APB2分频器设置为`/1`  
### keil5部分
LED闪灯代码：  
```c
while(1){
uint32_t i;
//点亮板载LED
HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_RESET);
//执行100万次for循环(for循环每次消耗8个指令周期，在8MHz下100万次大约耗时1s左右)
for(i=0; i<1000000; i++){}
//熄灭板载LED
HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_SET);
//再次执行100万次循环
for(i=0; i<1000000; i++){}
}
```
同时也要关闭代码优化  
通过Cubemx更改到HCLK最高时钟频率可以观察到LED运行速度的变化。  