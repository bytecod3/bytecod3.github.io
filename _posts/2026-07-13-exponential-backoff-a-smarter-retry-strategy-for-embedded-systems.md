---
title: "Exponential backoff: A smarter retry strategy for embedded systems"
date: 2026-07-13 00:00:00
categories: [EmbeddedTechnical]
---

## It's All About Resource Management
When writing embedded code, everything comes down to resource management. Resources in embedded are every and anything that plays a part in making your business and application logic work. Think RAM, FLASH memory, peripherals like UART, SPI, network and the like. 
All these resources are orchestrated in a controlled mannner to allow for a common output to be reached. That is all there is to embedded development. You read a sensor via SPI, save the data in memory, transmit the data to clients via a network interface. Done. Sounds easy. Right? 

## Design with reliabilty in mind
This is a hot topic sometimes in embedded, depending on the product being developed, but I will just brush it here. Think of a consumer product, it is built to be functional instead of reliable. Few try to make it reliable, because there are not consequences to it not being too reliable. A car's ABS is built to be functional *AND* reliable. If you need to break the car *HAS* to break.

Touching on consumer devices, sometimes the resources fail to cooperate. The RAM overflows, network fails to connect on time or drops, SPI fails to transmit a full frame of data. A good embedded developer should try their best to design the system to handle these cases. 

In this article I am going to show you how to handle resources that fail to connect on the first time, using a retry logic that uses a simple math formula.

## Traditional Retry Logic 
Let's take network connection for instance. Say you are trying to connect to some server somewhere. The traditional method is that you intitiate the reconnection and check the result. If connection has failed, you can *immediately retry* or *wait for a defined period of time* before trying to reconnect again. This is illustrated in the diagram below:

![traditional retry logic]({{"/assets/images/exponential-backoff-retry/traditional.png" | relative_url}})

It is clear that the retry interval never changes, remains the same throughout the retry periods. This is the simplest technique to implement. 

## Why is this not ideal
With a fixed retry interval, the MCU repeatedly wakes up and perfoms the same operation, even when the resource is unlikely to become available soon. For instance if the server needs 5 seconds to recover and you retry every 100ms, you have ideally performed 50 unnecessary retries. 

It suffices to say that most failures are temporary rather than permanent. For instance an SD card finishing a write cycle. Immediately retrying often results in repeated failures because the hardware is not ready yet. 

Immediately retrying to reconnect to WiFi or network yields the same results. 

In addition, if you have a complex system with multiple buses for example RS485, CANBUS, I2C, SPI, yet all of them are accessing the same resource, retrying at the same time will cause bus contention. Bus contention occurs when multiple devices or drivers attempt to use the same communication bus at the same time, leading to delays, collisions or failed transmission.

## Exponential Retry Logic
Gradually increase the time between multiple retries. This is the basic idea about exponential backoff. The diagram below explains better:

![exponential retry logic]({{"/assets/images/exponential-backoff-retry/exponential.png" | relative_url}})

The formula used is simple as well: 

```t_n = min(t_base x b^n, t_max) ```

where, ```t_base``` is the initial(base) delay, ```t_n``` is the delay before the nth retry, ```b``` is the backoff factor, usually 2, ```n``` is the retry attempt number(starting at 0). 

For example, suppose the base delay is 100ms and backoff factor is 2, with a maximum delay of 5 seconds. Here are the delays incrementally:

| Retry attempt | Formula | Delay |
|----------|----------|----------|
| 0    | 100 * 2^0   | 100 ms  |
| 1    | 100 * 2^1   | 200ms  |
| 2    | 100 * 2^2   | 400ms   |
| 3    | 100 * 2^3   | 800ms   |
| 4    | 100 * 2^4   | 1.6sec   |
| 5    | 100 * 2^5   | 3.2sec   |
| 6    | 100 * 2^6   | 5sec ( 6.4sec is capped at max 5 sec delay )  |


## Usage
There is a myriad of ways to implement this. I will show you a code snippet from one of my projects here: 

```c
 while(retry_count < MAX_RETRY_COUNT){

    fres = f_mount(&FatFs, "", 1);
    myprintf("Retrying to mount SD card...\r\n");

    if(fres == FR_OK) break;

    if(fres != FR_OK) {
        myprintf("Retrying to mount SD...retry delay (%lu)\r\n", retry_delay);
        retry_delay = get_min(1000 * 1 << retry_count, SD_CARD_MAX_RETRY_TIME); /* exponential back-off */
        HAL_Delay(retry_delay);

    } else {
        myprintf("f_mount status (%s)", sd_mount_status_to_name(fres));
        break;
    }

    retry_count++;
}

```

You can also use the math functions but I have basically used left-shifting to implement the formula. If curious, check how left-shifting and right-shifting are used to multiply and divide numbers.

## Conclusion

This can be improved further by far. For example, in systems with multiple buses(RS485, CANBUS) you can add intentional random jitter to the delay time to prevent the buses from ever reaching the maximum collision threshold. 

Exponential retry is an optional retry logic that can be used to make embedded systems more reliable and robust. For small hobby or simple projects, working with traditional retry for resources is okay. But if you want to design a robust and reliable system, you can never go wrong with exponential retry.