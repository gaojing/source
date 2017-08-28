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

## 1.6 Not running state
- the blocked state   
	1. Temporal(time related) events
	2. Synchronization event

- the Suspend state
- the Ready state

![](/images/freertos/freertos_state_trasition.jpg)

延时函数原型   
void vTaskDelay( portTickType xTicksToDelay );

## 1.7 the idle task and the idle task hook
void vApplicationIdleHook( void )

## 1.8 priority
void vTaskPrioritySet( xTaskHandle pxTask, unsigned portBASE_TYPE uxNewPriority )   
unsigned portBASE_TYPE uxTaskPriorityGet( xTaskHandle pxTask )

## 1.9 deleting a task
vTaskDelete()

## 1.10 the scheduling algorithm
- Prioritized Preemptive Scheduling

