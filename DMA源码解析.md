
源码：
1、对应步骤1，参考P102，P103的Table 8-2，配置DMA Configuration Data Structure如下：
```c
/* Configure DMA channel 0. Settings:
     * SRCADDR: address of the data to be copied (increasing).
     * DESTADDR: address the data will be copied to (increasing).
     * VLEN: use LEN for transfer count.
     * LEN: equal to the number of bytes to be transferred.
     * WORDSIZE: each transfer should transfer one byte.
     * TMODE: block mode.
     * TRIG: let the DMA channel be triggered manually, i.e., by setting the
     *       [DMAREQ.DMAREQ0] bit.
     * SRCINC: increment by one byte.
     * DESTINC: increment by one byte.
     * IRQMASK: disable interrupts from this channel.
     * M8: 0, irrelevant since we use LEN for transfer count.
     * PRIORITY: high.
     */
    dmaConfig0.SRCADDRH  = ((uint16)data >> 8) & 0x00FF;       //static char data[DATA_AMOUNT] = "DMA man trigger!";
    dmaConfig0.SRCADDRL  = (uint16)data & 0x00FF;             
    dmaConfig0.DESTADDRH = ((uint16)copy >> 8) & 0x00FF;      //static char copy[DATA_AMOUNT];
    dmaConfig0.DESTADDRL = (uint16)copy & 0x00FF;
    dmaConfig0.VLEN      = DMA_VLEN_USE_LEN;                 //#define DMA_VLEN_USE_LEN  0x00 :Use LEN for transfer count
    dmaConfig0.LENH      = (DATA_AMOUNT >> 8) & 0x00FF;      //#define DATA_AMOUNT 16
    dmaConfig0.LENL      = DATA_AMOUNT & 0x00FF;
    dmaConfig0.WORDSIZE  = DMA_WORDSIZE_BYTE;               //#define DMA_WORDSIZE_BYTE   0x00 :Transfer a byte at a time
    dmaConfig0.TMODE     = DMA_TMODE_BLOCK;                 //#define DMA_TMODE_BLOCK    0x01 :Transfer block of data (length len) after each DMA trigger
    dmaConfig0.TRIG      = DMA_TRIG_NONE;                   //#define DMA_TRIG_NONE      0 : No trigger, setting DMAREQ.DMAREQx bit starts transfer
    dmaConfig0.SRCINC    = DMA_SRCINC_1;                    // #define DMA_SRCINC_1        0x01 :Increment source pointer by 1 bytes/words after each transfer
    dmaConfig0.DESTINC   = DMA_DESTINC_1;                   //#define DMA_DESTINC_1      0x01 : Increment destination pointer by 1 bytes/words after each transfer
    dmaConfig0.IRQMASK   = DMA_IRQMASK_DISABLE;             //#define DMA_IRQMASK_DISABLE          0x00 :Disable interrupt generation
    dmaConfig0.M8        = DMA_M8_USE_8_BITS;              //#define DMA_M8_USE_8_BITS            0x00 :Use all 8 bits for transfer count
    dmaConfig0.PRIORITY  = DMA_PRI_HIGH;                  //#define DMA_PRI_HIGH                 0x02 :High, DMA has priority
```c

2、
```c
    /* The DMA configuration data structure may reside at any location in
     * unified memory space, and the address location is passed to the DMA
     * through DMA0CFGH:DMA0CFGL.
     */
    DMA0CFGH = ((uint16)&dmaConfig0 >> 8) & 0x00FF;
    DMA0CFGL = (uint16)&dmaConfig0 & 0x00FF;
```c

3、
```c
 /* Arm the DMA channel, so that a DMA trigger can initiate DMA writing,
    and apply 9 NOPs to allow the DMA arming to actually take effect. */
    DMAARM |= DMAARM_DMAARM0;  
    NOP();NOP();NOP();NOP();NOP();NOP();NOP();NOP();NOP(); // 9 NOPs
```c

```c
    // Trigger the DMA channel manually.
    DMAREQ |= DMAREQ_DMAREQ0;

    // Wait for the DMA transfer to complete.
    while ( !(DMAIRQ & DMAIRQ_DMAIF0) );
```c
    /* By now, the transfer is completed, so the transfer count is reached.
     * The DMA channel 0 interrupt flag is then set, so we clear it here.
     */
    
    // Clear interrupt flag by R/W0, see datasheet.
    DMAIRQ = ~DMAIRQ_DMAIF0;      
