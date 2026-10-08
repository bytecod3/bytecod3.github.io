---
title: "Telemetry Ingestion Pattern 1: Shared struct with MUTEX access"
date: 2026-10-8 00:00:00
categories: [CubeSat]
---

# Introduction

In one of the projects I have been doing on the side, I have been designing a temperature-reading rig with a few other data points that contain values such as battery voltage values, pressure points and the system uptime in seconds. The requirements are pretty strict in such a way that the system must report the latest data state for all the aforementioned data points. In this article I will explain to you how I design data (telemetry) collection and reporting architecture for this project.

It is helpful to know that there is a myriad of ways to capture and distribute such data across tasks in FreeRTOS. However to start on these patterns I will share with you one of the most commonly used patterns, the **shared struct pattern**. In following articles I will discuss other related patterns in this regard.

## The design problem

The major design problem in this project is how to capture the latest system state and update the processing/transmission task, and at the same time preventing a race condition.

As the title suggests, I use a single struct to store system data. Since this is a shared struct, one task can be updating the data points in the struct while the other task is reading. This should generally be avoided in good firmware. This writing and reading the same value at the same time causes what is known as a race condition.

## Race condition

A race conditions occurs when two or more tasks or threads access shared data concurrently, and as the result depends on the exact order in which they execute. For example, if one task is updating a shared variable while another task is reading it, the reading task may see the old value, the new value or an inconsistent intermediate state. This introduces unpredictable behaviour into the system, a behaviour that is fatal for in RTOS based embedded systems.

To help us prevent this, we use a MUTEX to protect or rather control access to what are known are CRITICAL REGIONS of the application.

### Why are MUTEXES necessary

The word MUTEX stands for MUtual EXclusion. Mutexes are necessary when multiple tasks can access the same shared resource and at least one of them can modify it. MUTEXes ensure that only one task can enter the protected section at a time. We will use a MUTEX to protect our shared struct.

By using a MUTEX, we exclude the shared struct from any atomic change while another task might be changing it.

## Solution

The diagram below shows my design architecture diagram:

![pattern-block]({{"/assets/images/telemetry-ingestion-shared-struct/shared-struct-block.png" | relative_url}})


As this diagram shows, it is well clear that we only have a single struct to handle all the data passing across the tasks. We have 2 producer tasks and a single consumer task. The producer tasks are responsible for data DAQ(data acquisition) from system sensors. To update the struct, they have to lock the struct first since it is shared. The code snippet below shows how one can implement this using a MUTEX.

````c++ 
/**
* @brief Shared struct pattern
* @author Edwin M.
*/


/// our shared struct type definition
typedef struct {
	uint32_t t_sec;
	uint16_t pressure;
	uint8_t temperature;
} SensorData_t;

/// we create a single shared struct variable
SensorData_t sensor_data;

SemaphoreHandle_t xSharedStructMutex;

static uint8_t createMutex(){
	xSharedStructMutex = xSemaphoreCreateMutex();
	if (xSharedStructMutex == NULL) return false;
	return true;
}

/// producer task
void task1(void* args){
	
	// initialise task variables here
	uint16_t p = 0;
	uint8_t t = 0;
	
	for(;;){
		// read sensors 
		t = readTemp();
		p = readPress();
		
		// update the shared struct 
		xSemaphoreTake(xSharedStructMutex, 0);
			sensor_data.pressure = p;
			sensor_data.temperature = t;
		xSemaphoreGive(xSharedStructMutex);
		
		// prevent idle task watchdog trigger
		vTaskDelay(pdMS_TO_TICKS(SOME_DELAY);		
	
	}
}

/// consumer task
void task3(void* args){
	
	// variable to hold a local copy of the shared struct
	SensorData_t sens_data;
	
	for(;;){
		
		// update the shared struct 
		xSemaphoreTake(xSharedStructMutex, 0);
			sens_data = sensor_data; // create a copy of the shared struct data
		xSemaphoreGive(xSharedStructMutex);
		
		// at this point we have a copy of the shared struct
		// and we have released the shared struct to other tasks 
		// so we can process all the data without worry 
		processSensorData();
		
		// prevent idle task watchdog trigger
		vTaskDelay(pdMS_TO_TICKS(SOME_DELAY);		
	
	}
}

````

In the consumer task you can see that we create a local copy of the shared struct by using:

````c++
// variable to hold a local copy of the shared struct
SensorData_t sens_data;
````

The access to shared struct should not be blocked for a long time, just like an interrupt service routine. What we can do is to immediately copy the struct int the local sensor variable and release the shared struct to other tasks, or at least this is how I do it. After this I can then process the data, an operation that can take time, without holding the shared struct captive.

### Major use case

The most common applications where I use this pattern is when I need to maintain and provide access to the latest state of a system, where I do not care about every individual sensor measurement over the past number of seconds, or when I do not need a time-correlated history of sensor samples.

For example, the telemetry struct can hold the most recently known temperature, pressure and other values, allowing tasks such as monitoring, logging, control logic or user interface to have the latest state, not the history.

### Known limitation

This pattern has a known limitation depending on how you want to capture the data. The shared struct represents the latest state, not a complete historical snapshot of the sensor measurements. Each field is updated independently as new measurements arrive. Suppose the struct contains:
```
temperature = 25.4 °C
pressure    = 101.2 kP
```

The data could have been captured at the following times:
```
Temperature - measured at 10:00:05
Pressure   - measured at 10:00:03
```

It is not a single snapshot of the system at a given point in time, say 10:00:05. When you need time correlated data, this is not the best pattern. Queue data sharing pattern is the most appropriate, which we will write about some other day.

## Conclusion
There you have it. Have fun!