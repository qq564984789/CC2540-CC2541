DMA的数据搬移：
- 内存和内存
- 外设和内存


先看Datasheet P95上的关于DMA的一段介绍：
The Direct Memory Access (DMA) Controller can be used to relieve the 8051 CPU core of handling data
movement operations, thus achieving high overall performance with good power efficiency. The DMA
controller can move data from a peripheral unit such as ADC or RF transceiver to memory with minimum
CPU intervention.
重点： DMA的作用就是为了减轻CPU的负担，它能在**内存和外设单元**直接传输数据而不经过CPU，这样更加提高系统效率。ADC/UART/RF收发器等外设单元和存储器之间可以直接在“DMA控制器”的控制下交换数据而几乎不需要CPU的干预。除了在数据传输开始和结束时做一点处理外，在传输过程中CPU可以进行其他的工作。这样，在大部分时间里，CPU和这些数据交互处于并行工作状态。因此，系统的整体效率可以得到很大的提高。

The DMA controller contains a number of programmable DMA channels for memory-memory data movement.
内存和内存数据的搬移

The DMA controller controls data transfers over the entire address range in XDATA memory space.
Because most of the SFR registers are mapped into the DMA memory space,



DMA的使用步骤：
1、In order to use a DMA channel, it must first be configured 。The DMA channel parameters must be configured before a DMA channel can be armed and activated。因此，我们的第一步是配置DMA channel的各个参数。
注意：The parameters are not configured directly through SFR registers, but instead they are written in a special DMA configuration data structure in memory. A DMA configuration-data structure may reside at any location decided on by the user software, and the address location is passed to the DMA controller through a set of SFRs,DMAxCFGH:DMAxCFGL.
首先必须配置DMA，但DMA的配置比较特殊：不是直接对某些SFR赋值，而是在外部定义一个结构体，对其赋值，然后再将此结构体的首地址的高8位赋给 DMAxCFGH，将其低8位赋给 DMAxCFGL。



2、Once a DMA channel has been configured, it must be armed before any transfers are allowed to beinitiated. A DMA channel is armed by setting the appropriate bit in the DMA channel-arm register DMAARM. Once a channel has been armed, the DMA controller reads the configuration data structure for that channel, given by the address in DMAxCFGH:DMAxCFGL.
方法：`DMAARM |= DMAARM_DMAARMx; `   意义：对 DMAARMx 赋值1，启用通道x的配置，使通道x处于工作模式。
注意：Note　that the time to arm one channel (i.e., get configuration data) takes nine system clocks;  thus, if the　corresponding DMAARM bit is set and a trigger appears within the time it takes to configure the channel, the　wanted trigger is lost。



3、When a DMA channel is armed, a transfer begins when the configured DMA trigger event occurs. 
注意：In addition to starting a DMA transfer through the DMA trigger events, the user software may force a DMA　transfer to begin by setting the corresponding DMAREQ bit. `DMAREQ |= DMAREQ_DMAREQx;`


备注：
Each DMA channel can be configured to generate an interrupt to the CPU on completing a DMA transfer.
This is accomplished with the IRQMASK bit in the channel configuration. The corresponding interrupt flag
in the DMAIRQ SFR register is set when the interrupt is generated.
Regardless of the IRQMASK bit in the channel configuration, the corresponding interrupt flag in the
DMAIRQ register is set on DMA channel completion. Thus, software should always check (and clear) this
register when rearming a channel with a changed IRQMASK setting. Failure to do so could generate an
interrupt based on the stored interrupt flag.


