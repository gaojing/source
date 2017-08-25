title: Using the FreeRTOS Real Time Kernel
categories: FreeRTOS
tags: [FreeRTOS]
---

# 1. Task Management

## 1.2 task function
task函数原型，不返回。

	void ATaskFunction( void *pvParameters )

## 1.4 CREATING TASKS
创建task

	portBASE_TYPE xTaskCreate( pdTASK_CODE pvTaskCode,
		const signed portCHAR * const pcName,
		unsigned portSHORT usStackDepth,
		void *pvParameters,
		unsigned portBASE_TYPE uxPriority,
		xTaskHandle *pxCreatedTask
	);




