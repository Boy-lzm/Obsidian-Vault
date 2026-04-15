1. Lin_SendFrame
	1. Lin_StopTransfer
		1. 关闭所有中断
		2. Uart_Lin_Drv_ClearTransmission：关闭OP_START
		3. Uart_Lin_Drv_WaitBusyClear：清除busy标志位
		4. Uart_Lin_Drv_SetIdleState：
			1. 如果当前的状态是send response，那么把模式切换回LIN_MODE（sendresponse采用软件模拟）
			2. 清除LSR和LISR
			3. 清空FIFO
			4. 如果为从机，那么把header done和 LIN error中断打开,把OP_MODE切换为HEADER，打开OP_START
			5. 如果是从机那么把HEADER DONE和LIN ERROR中断打开。
			6. 把当前的状态设置为之前的状态
			
	2. 记录checksum，data length，发送方向，数据指针。
	3. Uart_Lin_Drv_SendFrame：
		1. 判断当前的状态是不是SLEEP，是的话返回错误
		2. 判断当前状态是否是busy
		3. 如果是主机
			1. 记录当前的PID
			2. 如果是发送response，那么要记录数据，checksum，tx buffer和size
			3. 如果是接收数据，那么需要配置checksum和data length
			4. 将当前事件设置为：NO EVENT。当前状态设置为busy为TRUE，当前节点状态设置为break field end
			5. 清除两种模式下（header 和response）OP位
			6. 设置间隔场，PID和break长度
			7. 设置为master mode，设置OP MODE（header和response）和OPSTART
		4. 如果是从机
			1. 发送response（采用软件方式），关闭LIN MODE和MASTER，计算checksum和txbuff和tx size
			2. 打开 RBFI中断
			3. 将之前的状态设置为当前的状态，当前状态设置为RESPONSE SENT
			4. 状态设置为busy
			5. 清除OPSTART，打开fifo，清空fifo
			6. 发送第一个数据
			