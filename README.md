# AXI_DMA_FIFO
Transfer data from DDR memory to AXI4-Stream Data FIFO and back through AXI DMA
***
## Getting Start (Experiment Setup)
+ Windows 7/8/8.1/10 64-bit
+ Xilinx Vivado 2016/17/18.X (must include SDK)
+ [Board definition files](http://microzed.org/support/documentation/1519) for MicroZed 70X0
+ USB to micro USB cable x 1 
+ USB to mini USB cable x 1 (for JTAG)
+ RJ45 ethernet cable
+ [Putty.exe](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
## System Overview
本實驗將透過AXI DMA (Direct Memory Access)負責I/O Device與DDRX Memory之間的資料傳輸，其過程完全不需CPU(中央處理器)參與，CPU只需將傳送任務 (transfer task)和敘述位址(descriptors addresses)發送給DMA，就有更多時間將自己的運算單元與暫存器用在對其他程序的執行上。就PS端而言，以下分為兩種主流的傳送作業方式：
1. SG_Intr Mode 
   + Definition: Transfer packets in Interrupt Mode when AXIDMA core is configured in Scatter Gather Mode.
   + 由CPU完成的作業步驟包括：初始化DMA並設立傳輸，啟用DMA的中斷系統，初始化旗標，發送封包，待發生中斷時執行程式碼迴圈(ex: 再次指定傳輸，旗標值改變)，全部傳送完畢後跳開迴圈並檢查DMA傳送結果，指定發送下一次封包或關閉DMA的中斷系統。
2. SG_Poll Mode
   + Definition: Transfer packets in Polling Mode when AXIDMA core is configured in Scatter Gather Mode.
   + 由CPU完成的作業步驟包括：初始化DMA並設立傳輸，發送封包，檢查DMA傳送結果(可能重複數次直到傳送結束)，指定發送下一次封包。
## Lab 1: Create a New Zynq Project
