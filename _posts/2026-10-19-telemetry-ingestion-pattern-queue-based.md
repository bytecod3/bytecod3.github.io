---
title: "Telemetry Ingestion Pattern 2: Queue-based Ingestion"
date: 2026-10-19 00:00:00
categories: [CubeSat]
---

## Introduction
Welcome to the second part of telemetry ingestion patterns. In this article I am going to discuss queue-based telemetry ingestion. Queues are used as one of the main methods for inter-task communication, communicating data, events and results from one task to another. Queue-based ingestion is a relatively straight-forward method therefore I won’t spend so much time on it. Rather, I will show the practical use case and a few caveats when using queues.

## Catch up

Our goal in these telemetry ingestion articles is to understand how to propagate sensor data from one or multiple tasks to one or multiple other tasks, using practical design 
patterns

Examples in this article will use FreeRTOS, but the pattern can be borrowed into other embedded RTOS platforms as well.
## Queue definition

A queue is a data structure that uses the ***First In First Out(FIFO)*** pattern to store and access data. This means that the first element added to the queue is the first one to be removed, mirroring an actual queue in real-life. Elements are always inserted(enqueued) at the back and removed(dequeued) from the front. However, as we will see, it is possible to add items at the front in FreeRTOS, and to overwrite data that is already at the front of a queue.

## Queue creation

A queue can hold a finite number of fixed size data items. The maximum number of items a queue can hold is called its **length**.

For example, to create a queue that holds 10 integer items, using FreeRTOS we can create it as follows:

```c++
#define NUM_ITEMS (10)
QueueHandle_t exampleQueue;

uint8_t createQueue() {
	exampleQueue = xQueueCreate(NUM_ITEMS, sizeof(int));
	
	// returns NULL if not enough memory 
	// else the created queue is assigned to the queue handle
	if(exampleQueue == NULL) {
		return 0;
	} else {
		return 1;
	}	
}
```

## Sending data

Following the queue created above, we can then send the data to the queue using the xQueueSend() API call. xQueueSend() sends data to the back (tail) of the queue. An equivalent API call is the xQueueSendToBack().

In addition, we can also send data to the front(head) of the queue using the xQueueSendToFront(). An example code is shown:

```c++
// sending data to the back of the queue
BaseType_t sendStatus;

int value = 0;
sendStatus = xQueueSend(exampleQueue , &value, 0);

// from the documentation, this call expects the queue handle, pointer to the value to send, 
// and the ticks to wait before sending, in case the queue is full. In this case we do not wait 
```

Now, things are about to get pretty interesting. For sensitive data, it would be nice to always check if the queue sending call was successful or not. This is ***very*** useful when for example creating command queues/systems. For sensitive systems, you could do a simple check like this, to make sure that the command is not lost:

```c++
if (sendStatus != pdPASS) {
	// handle command delivery failure
	// maybe use exponential backoff here to retry sending 
}
```

I have personally used this technique to create command queues for example for PID control systems where the regulation must not fail, making the system more robust.

To be safe, if you care about your telemetry packets, always check if the send was successful, to have a visibility on the number of dropped packets for example.

When sending telemetry packets to a queue, it is helpful to know which is the optimal method to use. There are 2 ways to send data to queue:

1. By copy
2. By reference

### Queueing by copy

When we use xQueueSend(), FreeRTOS copies the specified number of bytes from the source variable to the queue’s internal storage. The queue’s item size is determined when we created the queue. An example is shown below:

```c++
{
	int value = 34;
	xQueueSnd(exampleQueue , &value, 0);
	
	// check for successful send here
	
	// at this point, the queue creates its own local copy of "value". So 2 values exist
	// one is the sender's, owned by the sender
	// the other is the queue's, owned by the queue
}
```

Every silver lining has a cloud: The disadvantage of this method is that the memory usage is high. If for example you are sending very large structures like this one below, you end up creating a huge memory deficit. This would not be a good design.

```c++
typedef struct {
    // Temperature sensors
    float temperature_top;
    float temperature_middle;
    float temperature_bottom;
    float temperature_surface;
    float temperature_ambient;

    // Battery measurements
    float battery_voltage;
    float battery_current;
    float battery_temperature;
    uint8_t battery_soc;
    uint8_t battery_status;

    // Heater and control system
    float heater_output;
    float temperature_setpoint;
    float pid_error;
    float pid_integral;
    float pid_derivative;

    // System health
    uint32_t uptime_seconds;
    uint32_t free_heap_bytes;
    uint32_t stack_high_watermark;
    uint32_t watchdog_resets;
    uint32_t sensor_fault_flags;

    // Communication statistics
    uint32_t mqtt_messages_sent;
    uint32_t mqtt_messages_dropped;
    uint32_t wifi_reconnects;

    // Metadata
    uint32_t sequence_number;
    uint32_t timestamp_ms;

} Telemetry_t;
```

### Queueing by reference

When queueing by reference, the only thing we send to the queue is the pointer to a data structure instead of copying the entire data structure into the queue. I normally use this technique when I want to exchange large amounts of data such as the huge struct shown above.

The major problem that queueing by reference solves is that if the struct is large, say 1KB of a packet, and you queue by copy, say the queue holds 10 items, that is 10KB, gone.

With queueing by reference, the queue stores only a pointer to the 2KB buffer. The queue does not contain the actual data, it contains an address that tells the receiving task where the data resides. I will now give you an example:

```c++
#define SAMPLE_COUNT (1024)
typedef struct {
	uint32_t timestamp_ms;
    float temperature;
    float voltage;
    float current;

    uint16_t raw_samples[SAMPLE_COUNT];
} Telemetry_t;

QueueHandle_t telemetryQueue;

// create the queue
telemetryQueue = xQueueCreate(
	5, 
	sizeof(Telemetry_t*)
);

// sizeof(Telemetry_t*) tells FreeRTOS that each queue item is a pointer, not the entire
// telemetry structure 

```

The sending task sends data as follows:

```c++
Telemetry* telemetry = fetchTelemetryBuffer();// MUST return a valid memory location 

// populate the telemetry buffer here

if(xQueueSend(telemetryQueue, &telemetry, 0) == pdPASS) {
	// send success
	// the receiver task own the telemetry buffer
} else {
	// send failed
	// the sender task still owns the buffer
}
```

Note that &telemetry is the address of the pointer to Telemetry. So the queue stores addresses, as mentioned earlier.

The receiving task then receives the data as follows:

```c++
Telemetry_t* receivedTelemetry;

if(xQueueReceive(
	telemetryQueue,
	&receivedTelemetry,
	portMAX_DELAY) == pdPASS) {
		// deference and process the data
	
	}
```

receivedTelemetry is a local pointer variable that will now store the address of the data buffer.

Again, every silver lining has a cloud. Do not forget to release the memory. This will be a discussion for another article. Also, queuing by reference does not automatically provide a consistent snapshot. If one task modifies the buffer while another reads it, the receiver could observe a mixture of old and new values. Ownership transfer or suitable synchronization is necessary. You can use a MUTEX here.

## Conclusion

That’s it. For this article, a high level introduction to using queues was introduced. You might find patterns using multiple queues to send data to multiple tasks, from a single task, a design I can write about later. There are several techniques I have not discussed here, just to keep the article readable and short . Otherwise, the next queue patterns will build upon this one.

Keep building, and see you next time!