---
title: Project Proposal
layout: default
parent: wastebin
nav_order: 1
---

# Project proposal

The idea is to create a waste bin that recognizes if the waste that is thrown into it belongs into this bin.

After the user throws trash into the bin, it recognizes if the piece belongs into the bin.
If the waste belongs into the bin, it is thrown into the collecting part of the bin.
However, if it does not belong into the bin, it gets thrown back at the user and makes a loud sound.

A malfunction would be if wrong waste is accepted or if correct waste is thrown back at the user.

The device is uncomfortable for people throwing in the wrong kind of trash and getting trash thrown at them by the bin and being publicly shamed for not recycling properly.

## Implementation

A trash bin is equipped with a camera and checks if the trash about to be thrown in is the correct kind of trash. For this, an AI might be used, that runs on an Arduino. If it is the wrong kind of trash, a catapult activates inside the bin and tries to throw the item back out of the bin and additionally makes a loud noise using a speaker.
If it is the correct kind of trash, the catapult moves down, so that it stays in the bin.

Depending on the time available and the overall results with training the AI, it might be replaced with a Wizard of Oz prototype where a button is pressed depending on the needed action.

## References
Dataset for AI Model for plastic trash recognition:
[https://github.com/AgaMiko/waste-datasets-review](https://github.com/AgaMiko/waste-datasets-review)

Object detection with Arduino:
[https://www.teachmemicro.com/detect-objects-camera-arduino/](https://www.teachmemicro.com/detect-objects-camera-arduino/)

Wooden Catapult:
[https://www.instructables.com/Arduino-Controlled-Catapult/](https://www.instructables.com/Arduino-Controlled-Catapult/)

weight measurement with arduino:
[https://prilchen.de/eine-waage-mit-arduino-erstellen/](https://prilchen.de/eine-waage-mit-arduino-erstellen/)


## Sketch
![FirstSketch](assets/ersteSkizze.png)

## Time plan

| Description | Due date |
| ----------- | -------- |
| Test catapult mechanism | 30. November |
| Integrate catapult in bin | 10. December |
| Flap downwards | 17. December |
| Integrate Camera and Button | 3. January |
| Add Sound | 10. January |


## List of materials

| Description | Source | Price |
| ----------- | -------- | -------- |
| wood plates | we have it | - |
| 3D-print Filament | we have it | - |
| Web Cam | [MediaMarkt](https://www.mediamarkt.at/de/product/_trust-webcam-exis-mit-mikrofon-schwarz-17003-1151054.html) | 5,99€ |
| Speaker | we might have one | - |
| 8m Wire | set  | - |
| Button x3 | set  | - |
| Servomotor x5 | [Amazon](https://www.amazon.de/AZDelivery-Servo-Mikro-Servomotor-Metallgetriebe-kompatibel-Arduino/dp/B086V3VP72/ref=sr_1_6?keywords=Arduino+Servo&qid=1669576800&sr=8-6)  | 18,65€ |
| spring | we might have one  | - |
| cord | we have it  | - |
| battery | we have it  | - |
| Rubber band | we have it  | - |
| Gaffer | we have it  | - |
| Screws | we have it  | - |
| Breadboard | set  | - |
