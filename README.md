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
+ [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
## System Overview
本實驗將透過AXI DMA (Direct Memory Access) 負責I/O Device與DDRX Memory之間的資料傳輸，其過程完全不需CPU(中央處理器)參與，CPU就有更多時間用在軟件執行上，由CPU完成的作業步驟包括：設立傳輸，啟用中斷，產生中斷時執行程式碼。
## Lab 1: 
