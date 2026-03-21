# 简介

ARM公司开发的一套软件标准接口，为基于ARM Cortex-M处理器的微控制器提供一致的开发接口。CMSIS通过定义一组标准化的API和库函数，极大地简化了嵌入式软件开发的复杂性，提高了代码的移植性和重用性。

# CMSIS标准软件架构

CMSIS标准的软件架构主要分为以下四层：用户应用层，操作系统层，CMSIS层以及硬件寄存器层
![[Pasted image 20260322005130.png]]其中CMSIS层起着承上启下的作用，一方面该层对硬件寄存器层进行了统一的实现，屏蔽了不同厂商对Cortex-M系列微处理器核内外设寄存器的不同定义，另一方面又向上层的操作系统和应用层提供接口，简化了应用程序开发的难度，使开发人员能够在完全透明的情况下进行一些应用程序的开发。也正是如此，CMSIS层的实现也相对复杂，下面将对CMSIS层次结构进行剖析。

- CMSIS-Core（最核心）
1. 定义内核寄存器寄存器地址和访问函数。无论使用 STM32 还是 NXP，操作 NVIC 的函数名都是一样的（NVIC、SysTick等）
2. 提供访问内核的标准API
3. 启动文件（startup_xxx.s）
4. 中断向量表
5. 编译器适配: 屏蔽了 Keil、IAR、GCC 之间的语法差异。例如，__INLINE、__WEAK 等宏定义,让一套代码可以在不同编译器下编译

- CMSIS-Driver( 外设驱动层)

定义了常见片上外设（如 I2C、SPI、UART、以太网、USB）的统一驱动接口。

- CMSIS-RTOS（操作系统接口）

1. 解决的问题：不同的 RTOS（FreeRTOS、RTX、ThreadX）API 接口各不相同
2. CMSIS-RTOS v2 定义了统一的标准 API（如 osThreadNew、osMutexAcquire）

Additional:

- 核内外设访问层（CPAL，Core Peripheral Access Layer）：

该层由ARM负责实现。包括对寄存器名称、地址的定义，内核寄存器、NVIC、调试子系统的访问接口定义以及对特殊用途寄存器的访问接口（例如：CONTROL，xPSR）定义。由于对特殊寄存器的访问以内联方式定义，所以针对不同的编译器ARM统一用__INLINE来屏蔽差异。该层定义的接口函数均是可重入的。

- 片上外设访问层（DPAL,Device Peripheral Access Layer）：

该层由芯片厂商负责实现。该层的实现与CPAL类似，负责对硬件寄存器地址以及外设访问接口进行定义。该层可调用CPAL层提供的接口函数同时根据设备特性对异常向量表进行扩展，以处理相应外设的中断请求。

- 外设访问函数（AFP,Access Functionsfor Peripherals）：

该层也由芯片厂商负责实现，主要是提供访问片上外设的访问函数，这一部分是可选的。

对一个Cortex-M微控制系统而言，CMSIS通过以上三个部分实现了：

- 定义了访问外设寄存器和异常向量的通用方法；
- 定义了核内外设的寄存器名称和核异常向量的名称；
- 为RTOS核定义了与设备独立的接口，包括Debug通道。