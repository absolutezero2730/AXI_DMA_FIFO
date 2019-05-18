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
本實驗將透過[AXI DMA (Direct Memory Access)](https://www.xilinx.com/support/documentation/ip_documentation/axi_dma/v7_1/pg021_axi_dma.pdf) 負責I/O Device與DDRX Memory之間的資料傳輸，其過程完全不需CPU(中央處理器)參與，CPU只需將傳送任務 (transfer task)和敘述位址(descriptors addresses)發送給DMA代理，本身就有更多時間將運算單元與暫存器用在對其他程序的執行上。就PS端而言，典型的的傳送作業方式有兩種：
1. SG_Intr Mode 
   + Definition: Transfer packets in Interrupt Mode when AXIDMA core is configured in Scatter Gather Mode.
   + 由CPU完成的作業步驟包括：初始化DMA並設立傳輸，啟用DMA的中斷系統，初始化旗標，發送封包，待發生中斷時執行程式碼迴圈(ex: 再次指定傳輸，旗標值改變)，全部傳送完畢後跳開迴圈並檢查DMA傳送結果，指定發送下一次封包或關閉DMA的中斷系統。
2. SG_Poll Mode
   + Definition: Transfer packets in Polling Mode when AXIDMA core is configured in Scatter Gather Mode.
   + 由CPU完成的作業步驟包括：初始化DMA並設立傳輸，發送封包，檢查DMA傳送結果(可能重複數次直到傳送結束)，指定發送下一次封包。

Block diagram (Compatible with the above two Modes)：
![image](https://github.com/absolutezero2730/AXI_DMA_FIFO/blob/master/Design%20overview.png)
For high speed computing design, also for reducing the payload on CPU, the DMA may be the most efficient way of doing so. Why DMA? If we had data coming in from a very fast ASIC readout chip, or a multi-channel ADC device, and we need to store it very quickly through the FPGA to the DDR memory. We can't just rely on the processor to transfer data. This may overkill the intelligent performances of CPU and waste too much of its registers. 

In this tutorial we are using the DMA interface to build a simple data transfer through PL to the DDR memory. The AXI DMA and AXI Data FIFO are connected through the AXIS_MM2S and AXIS_S2MM buses. These two AXIS buses mainly source and sink data stream without address. The processor will communicate through the AXI-lite bus to the DMA for setting up, initiating and monitoring. The AXI_MM2S and AXI_S2MM are memory-mapped AXI buses that connect to the memory controller. 

When this experiment is complete, you will be able to: 
1. Use [AXI4-Stream Data FIFO](https://www.xilinx.com/support/documentation/ip_documentation/axis_infrastructure_ip_suite/v1_1/pg085-axi4stream-infrastructure.pdf) and [AXI DMA](https://www.xilinx.com/support/documentation/ip_documentation/axi_dma/v7_1/pg021_axi_dma.pdf)
2. Use the xaxidma driver on the Xilinx AXI DMA to transfer packets in polling mode
3. Use the xaxidma driver on the Xilinx AXI DMA to transfer packets in interrupt mode
4. Get DMA status through UART
## Lab 1: Create a New Zynq Project
1. Launch Vivado.
2. Select <b>File</b> -> <b>New Project</b> or click on <b>Create New Project</b> under Quick Start. 
3. Click the browse icon. Browse to set the Project location to your desired project location and <b>click Start</b>.
4. Set the project name to <b>Microzed_7020_AXI_DMA_test</b>. Also verify the <b>Create project subdirectory</b> check box is selected. Click <b>Next ></b>.
5. The project will be RTL based. Leave the radio button for RTL Project selected. Since this is a brand new project, check the box for <b>Do not specify sources at this time</b>. Click <b>Next ></b>.
![](https://github.com/absolutezero2730/AXI_DMA_FIFO/blob/master/catch01.PNG?s=2)
6. In the Select area, select <b>Boards</b>.
![](https://github.com/absolutezero2730/AXI_DMA_FIFO/blob/master/catch02.PNG|width=1)
7. Single-click the <b>MicroZed Board</b> that matches your configuration. Click <b>Next ></b>.
8. A project summary is displayed. Click <b>Finish</b>. The Vivado cockpit is now displayed. 
## Lab 2: Create and Edit a Block Design
1. The recommended way to add an embedded processor is through the Block design method via IP Integrator. Select <b>Create Block Design</b>. 
2. Give the Block Design a name (default: design_1). Click <b>OK</b>.
3. In the Diagram window, click the <b>Add IP</b> text icon. 
4. Find the [<b>ZYNQ7 Processing System</b>](https://www.xilinx.com/support/documentation/ip_documentation/processing_system7/v5_5/pg082-processing-system7.pdf) IP. Either double click this or drag and drop to the Diagram window. 
