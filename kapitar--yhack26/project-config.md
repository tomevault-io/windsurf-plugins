---
trigger: always_on
description: we are building this idea
---

we are building this idea

Mission: LeadMe is the next-generation, AI-powered, self-navigating white cane for the visually impaired.
LeadMe
Why this should be done:
* Guides dogs costs 50k - 60k + 2 years of training
* Guide dogs havea 7-10 year career span, users need to constantly restart and relearn with a new dog
* Guide dogs are high maintenance, needs grooming, care, exercise
* Business often illegally deny access guide dogs
* Guide dogs gets distracted a lot
* It has a big ass market size
   * Smart cane market
      * 2024 valuation: 400-780 mil
      * 2032 projection valuation: 1.2 bil
* Investors are willing to fund it
   * __https://www.leadsontrees.com/news/innovative-tech-for-the-visually-impaired-wewalk-secures-10.14-million-in-funding-to-transform-mobility__
 Solution:
* Basically the 21st century robotic successor to the guide dog. It turns a white cane into an active navigational partner, giving users the same features of a guide dog but in a tiny, accessible form factor.
Basically:
 At the tip of the cane is a motorized 3-wheel kiwi drive for homonymic movement. The motors create directional intent, meaning the user feels a physical tug or push through the handle that leads them.
Features:  Does basically everything a guide dog does. So like this is what chat said a guide dog does
* Active Guidance: Keeps a straight line and pulls you "left" or "forward" on command.
* Level Changes: Detects curbs and stairs, stopping to alert the user.
* Overhead Safety: Senses head-level hazards like tree branches or signs.
* Target Finding: Identifies and leads users directly to elevators, front desks, bus stops, or empty seats.
* Intelligent Disobedience: Uses active resistance to physically stop you from walking into danger like traffic or a ledge. 

we have the hardware, give me the idea of how to implement these features

Show less
9:19 AM
This is a genuinely exciting project. Here's how I'd break down the software/sensor implementation for each feature, given you have the hardware (kiwi drive tip, handle with haptic feedback, and presumably a sensor suite on the cane):

Sensor stack you'll likely need:

A depth camera (like Intel RealSense or similar) for the primary scene understanding, a LiDAR or ultrasonic sensors for reliable distance measurement, an IMU for orientation and movement, GPS + compass for outdoor navigation, a camera for visual recognition tasks, and a microphone/speaker for voice interaction. Most of these can be mounted along the shaft with the compute unit (Raspberry Pi 5 or Jetson Nano) in the handle.

Let me diagram the full system architecture and then break down each feature:



Now let's go feature by feature:

Active guidance — straight-line keeping and directional tugs

This is fundamentally a path planning + closed-loop motor control problem. You fuse GPS outdoors with a pre-loaded map (OpenStreetMap works well) or indoor beacon/WiFi positioning indoors. A* or Dijkstra gives you the route. The kiwi drive tip gets PID-controlled to constantly apply a gentle corrective force — if the user drifts left of the intended path, the motors apply a rightward bias that the user feels as a tug through the handle. The key insight is that you're not trying to steer the user rigidly; you're creating a "magnetic" pull toward the correct heading that they can feel and follow naturally. Force magnitude should scale with deviation angle.

Level changes — curbs and stairs

Your depth camera (pointed slightly downward at a ~20° angle) generates a point cloud of the ground plane ahead. Sudden depth discontinuities — a drop of more than ~5cm or a rise more than ~3cm within a short horizontal span — trigger an alert. For stairs, you look for repeating parallel discontinuities at regular intervals (~17–18cm riser height). The cane stops and pulses the handle vibration motor in a distinct pattern (two short pulses = step up, one long = step down). This is well-solved territory — you can adapt the algorithm from Microsoft's Seeing AI or existing smart cane research.

Overhead safety — branches and signs

This is the trickiest one because traditional canes only cover the ground. You'll want a second ultrasonic or IR sensor array mounted higher on the shaft, pointed forward and slightly upward, scanning the 1.2m–2.1m height band (head-level). A simpler approach: tilt the depth camera upward in software by processing only the upper portion of the depth frame. When an object is detected in that zone within ~1.5m, a rapid vibration pattern fires. The challenge is false positives (awnings, doorframes) vs actual hazards — you'll need a classifier that distinguishes passable overhead structures from collision risks using size and shape heuristics.

Target finding — elevators, bus stops, seats


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kapitar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
