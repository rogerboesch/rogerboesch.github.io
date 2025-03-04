---
layout: post
---

## 🚀 Eyes in the Sky: A Drone That Sails with Me

### Introduction

Imagine sailing through the open sea while your drone autonomously follows your boat, providing an eagle-eye view without any manual control. Or, as you approach a marina, the drone flies ahead to check for an available berth. These were the key use cases that led me to develop a Proof of Concept (PoC) integrating an ArduPilot-controlled drone with the NMEA 2000 network on my sailing boat. In this post, I will take you through the technical journey of building this system, the challenges I faced, and the remarkable results achieved.

![Waypoints](/images/ardupilot-waypoints.png)

### The Concept

The goal was to develop an autonomous drone system that would:

- Fly relative to the boat – The drone would follow the boat at a set altitude and distance, adjusting dynamically to its movements.

- Scout marina berths – The drone could fly to pre-stored GPS locations in a marina to check which berths were free.

- Navigate to any saved waypoint – It could autonomously fly to waypoints stored in my NMEA 2000 log, providing an aerial view of areas of interest.

To achieve this, I needed a robust autopilot system, real-time communication between the drone and the boat, and reliable GPS integration.

### Hardware Setup

**Drone Selection & ArduPilot Integration**

I built the drone using an ArduPilot-compatible Pixhawk flight controller, which offered full MAVLink support and allowed flexible customization. The drone was a quadcopter design with the following key components:

- Frame: 500mm quadcopter frame (Holybro)
- Flight Controller: Pixhawk 4 (running ArduPilot)
- Motors & ESCs: T-Motor MN2212 with 30A ESCs
- GPS Module: uBlox M8N (for precise positioning)
- Telemetry: 433MHz telemetry module for communication
- Camera: Gimbal-stabilized HD camera for live video feed

**NMEA 2000 Integration**

To integrate the drone with my sailing boat’s NMEA 2000 network, I needed a way to bridge the MAVLink protocol (used by ArduPilot) with the NMEA 2000 messages. I achieved this using:

- Raspberry Pi as the Middleware – Connected to the boat’s NMEA 2000 network via an Actisense NGT-1 USB gateway.

- Custom Rust Code – Running on the Raspberry Pi, this converted NMEA 2000 GPS and heading data into MAVLink commands.

## Software Development

**MAVLink & NMEA 2000 Communication Bridge**

I wrote a Rust program that continuously read GPS and heading data from the boat’s NMEA 2000 bus and translated it into MAVLink messages for the drone. This allowed the drone to stay in relative position to the boat dynamically.

Key parts of the code:

- Reading NMEA 2000 GPS coordinates.

- Translating these into a MAVLink SET_POSITION_TARGET_GLOBAL_INT message.

- Sending periodic updates to adjust for wind and drift.

**Pre-Stored GPS Tracks for Marina Berths**

To enable the drone to fly to specific marina berths, I logged GPS coordinates of various dock spaces into a database. When needed, the drone can be commanded to fly autonomously to any of these points and relay back real-time video.

The workflow:

- User selects a berth number on a mobile app.

- The Raspberry Pi retrieves the GPS coordinates from the database.

- MAVLink commands are sent to the drone to fly to the target waypoints.

- The drone streams live video, allowing me to see if the berth is occupied.

**Autonomous Waypoint Navigation**

I also extended the system to allow the drone to fly to any waypoint stored in my boat’s navigation log. This feature is useful for checking anchor spots, navigating complex waterways, or scouting ahead while sailing.

Challenges & Solutions

- GPS Latency & Drift
    
    Issue: The drone had minor lag in adjusting its position relative to the boat, especially at high speeds.

    Solution: Implemented Kalman filtering to smooth GPS updates and increased update frequency to minimize lag.

- NMEA 2000 to MAVLink Compatibility

    Issue: Direct translation between NMEA 2000 and MAVLink was not straightforward.

    Solution: Used a middleware (Raspberry Pi) to handle data conversion and ensure synchronized messages.

- Wireless Communication Reliability

    Issue: Over longer distances, Wi-Fi connectivity between the drone and boat became unstable.

    Solution: Implemented a dual-communication system using 433MHz telemetry.

## Results & Future Enhancements

The final PoC worked remarkably well:

- ✅ The drone maintained its position relative to the boat, dynamically adjusting as the boat moved.

- ✅ It successfully flew to pre-stored marina berths and relayed real-time video.

- ✅ It could navigate to any saved waypoint, providing a powerful scouting tool for sailing adventures.

### Future Plans

- 🚀 Integrate LiDAR for better altitude stabilization over water.
- 🚀 Improve obstacle avoidance for automated docking.
- 🚀 Implement AI-based image recognition to detect free berths automatically.

### Conclusion

This project demonstrated the power of integrating drones with maritime navigation systems. The ability to have an autonomous drone assist in navigation, surveillance, and scouting tasks significantly enhances the sailing experience. With further refinements, this technology could become an essential tool for sailors and maritime explorers alike.

If you’re interested in building a similar system or have ideas for enhancements, let’s discuss in the comments!

