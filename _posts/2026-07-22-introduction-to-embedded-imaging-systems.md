---
title: "Introduction to Embedded Imaging Systems"
date: 2026-07-22 00:00:00
categories: [EmbeddedTechnical]
---

## Imaging systems in general 
This is going to be a short introduction article just to satisfy my curiosity in embedded imaging systems. I acknowledge that imaging is an extremely broad topic with books written all over about it, so a simple artice is not going to do it any justice. 

Imaging systems range from modern computer vision applications, to satelite hyperspectral imaging, high-end cameras and lens systems, self driving cars - the list is endless. 

I have picked just a grain of this area and I will focus on low level embedded imaging systems, as this article is inspired by my cubesat camera system.

## Its necessity in embedded systems
I think that if we could create our own artificial eyes, coupled with all the nervous system accessories that are exactly identical to our eyes, then we would unblock an area of technology that has been a headache for decades. An imaging system is not only an extension of our sight and vision, but also a way of transferring what we see into a digital form (think of a second image processing brain, only artificial)

Embedded applications most times will want to have a way of visually representing the world around the device, satelite, vehicle etc. What you will find out is that most embedded systems are never standalone systems anyway, they are ingrained into the belly of a larger system. 

For instance, visual-light cameras see the world in much the same way as humans, so they are often used in autonomous vehicles to identify traffic signs, lane lines, and other markings that would be difficult or impossible for LiDAR or radar to capture. Cameras are crucial when it comes to classifying objects into categories like cars, trucks, pedestrians or cyclists. 

Another interesting application is the use of cameras is in industrial production lines. These types of cameras are known as *closed system cameras* simply because most come already built with specific application softwares tied to them. For example barcode readers, vision sensors, inclinition cameras, ideentification cameras etc. The biggest advantage of these cameras is the ease of used. However, these cameras do not allow for a lot of flexibilty in terms of customisation. 

Then there are *open system cameras*. This type is programmable. This typically means that the user can tailor the camera to meet their own specific need and application. Mostly it comes with Linux, offering a great ease of driver development. 

![imaging systems]({{"/assets/images/introduction-to-embedded-imaging-systems/camera.png" | relative_url}})

## Communication standards used
When designing for camera systems, it is crucial to understand the communication protocols used to transfer bit data from the camera modules to processing and storage. Options like GigE for industrial cameras and PCIe/MIPI for embedded systems are some critical connection methods.

Most common camera interfacing standards are the following: This list is not inclusive of some proprietary protocols

| Protocol | 
|----------|
| MIPI-CSI-2   | 
| FPD-Link    | 
| USB-3.0   | 
| MIPI A-PHY   | 
| GMSL     |  

The choice of interface affects the achievable resolution and frame rate(bandwidth), the maximum cable length between the camera and host, and the complexity of system design. I will summarize the protocols down below.

### MIPI CSI-2(Mobile Industry Processor Interface, Camera Serial Interface 2)
Widely used especially in mobile and embedded devices. It provides bandwidths of ~2.5Gps per lane over short distances(< 30cm>). It is majorly used for direct sensor-to-SoC connections.

### FPD-Link III
This is a serializer-deserializer protocol from Texas Instruments for long-distance high-speed video transmission(up tp ~4.16 Gps forward bandwidth). It can reach up to ~15m over COAX or STP cable. It is often used in automotive ADAS cameras, industrial and robotics systems.

### GMSL (Gigabit Multimedia Serial Link) - Extending MIPI over COAX
This is also a family of serializer-deserializer from Analog Devices for automotive and high end cameras. GSML supports up to ~6 Gbps forward bandwidth and ~15m cables. It carries video, power and control over a single cable.

### MIPI A-PHY
This is from the MIPI alliance for transmitting CSI-2 and DSI signals over 15m or more. It supports up to 16Gbps in one direction, using COAX or STP cables. It also has error correction. 

### FPD-Link IV
Ideal for advanced ADAS and high resolution sensors in robotics. It offers up to 8 Gbps per link, better EMI resistance. Suitable for next gen cameras.

### Thermal Camera Interfaces 
These cameras often output video in non-standard formats. SOme provide analog vide (NTSC/PAL) or parallel digital output, requiring interface bridges or custom capture boards. Others use USB or SPI interfaces.

## Conclusion
This short article brushed on some generic imaging systems in embedded systemsm, also introducing some common protocols used in cameras for embedded systems. Probably a further article will discuss these protocols in detail. Stay Curious!