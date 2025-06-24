---
layout: default
title: Lost Finder Bot
---

# Lost Finder Bot  

## Introduction

Several people get lost in unknown surroundings every year. A recent study done by Yosemite National Forest Search and Rescue showed that, in the United States alone, 4,661 people per year were lost in the woods and required assistance to get back to safety. Furthermore, it is estimated that 50,000 search and rescue missions are conducted each year nationwide. Many studies have shown that people who are lost also tend to walk around in circles without even noticing, severely hindering their chances at finding their way back to where they started. In large expanses like forests, search operations often require extensive manpower or costly resources such as helicopters. In hazardous environments such as fire or war zones, it may be perilous for humans to conduct thorough searches to locate missing individuals.

## Motivation

This project aims to serve as a proof of concept that a fleet of low-cost, autonomous robots can be designed and implemented to aid in search and rescue missions for people who are lost in environments, especially those that are dangerous or inconvenient for humans to explore. This could include lost children or hikers in the woods, people trapped in environments too dangerous for humans to explore, or even elderly people with dementia who wander and need to be found again. The machine learning algorithms implemented on this robot could be further trained to better suit whatever environment it is intended to be deployed in. It must be able to complete the following tasks:

- **Environment exploration**: Randomly explore its environment without colliding with any obstacles.  
- **Keyword Spotting**: Detect an audio cue/call for help.  
- **Object detection**: Search in its immediate surroundings for any humans.  
- **Navigation**: Move towards a human-like figure until it is certain that it is in fact a person.  
- **Communication**: Relay the location of the person to mission control and call for backup.

## Dataset Details

For this project, I designed and trained two distinct models. The first is a **keyword spotting model**, which detects the words "help," "stop," and "start." Two other categories were predicted: silence and unknown. The second is a **person detection model**, which takes in a 160x120 grayscale image and returns whether or not there is a human in the frame.

For the keyword spotting model:
- 50 audio samples of each word were recorded.
- The loudest sections were extracted and converted to `.wav` files.
- Audio was augmented with background noise, pitch shifting, and time stretching, improving accuracy by ~10%.
- For unknown values, Pete Warden’s keyword spotting dataset was used (e.g., “bed,” “bird,” “cat,” etc.).

For the person detection model:
- 118 images were taken with the XIAO ESP32S3 camera in three different environments.
- Images included various human poses and background conditions, and were scaled to 160x120 grayscale.
- Augmentation was done via flipping and brightness/contrast adjustments.
- The dataset was supplemented by Konstantin Werner’s Kaggle human detection dataset.
- Initial manual bounding box labeling was later abandoned due to memory constraints.

## Model Design

### Keyword Spotting

The KWS model used the `tiny_conv` architecture:
- A single depthwise separable convolutional layer
- A fully connected layer
- A softmax output with 5 categories
- Trained using cross-entropy loss

### Person Detection

Initially, I attempted a **bounding box detection model** that ran directly on the microcontroller:
- Combined classifier + regressor using a shared feature extractor
- Feature extractor: 3× depthwise separable conv pairs
- Classifier: Additional 3× conv pairs + global average pooling + softmax output
- Regressor: Same structure but with larger filters + smooth L1 loss
- Size: ~95 kB post-quantization, but needed 6MB arena, which was too large

This model was eventually replaced with a **cascade system**:
- Microcontroller runs a lightweight binary classifier with:
  - 4× depthwise separable conv pairs
  - Global average pooling → Dense(64) → Dropout → Sigmoid output
- If a person is detected, image is sent via TCP to a server
- Server uses TensorFlow's MobileNetV2 SSD to return bounding box
- Robot updates heading and continues toward the target

## Model Training and Evaluation

### Keyword Spotting

- Trained with TensorFlow’s `speech_commands` module
- Accuracy:
  - Training: 90%
  - Test: 92.5% (92.48% after quantization)
- Size:
  - Original: 102 kB
  - Quantized: 23 kB
- Training: 3000 epochs (2000 @ 0.001 learning rate, 1000 @ 0.0001)

### Person Detection

- Accuracy:
  - Training: 98%
  - Validation: 81%
  - Test: 79%
- Post-quantization:
  - Size: 295 kB → 34 kB
  - Test accuracy: ↑ to 81.4%
- Model prefers false positives over false negatives (desired for rescue)
- Performed well on real-world XIAO images despite broader Kaggle dataset variability

## Deployment / Hardware Details

The robot includes:
- **Motors**: 2× servos + 2× ball transfer wheels
- **Board**: XIAO ESP32-S3 with OV2640 camera and digital mic
- **Sensors**: IR sensor, IMU, GPS
- **Control**: PID heading correction using IMU, collision detection via IR

### Robot State Machine

1. **STOPPED**: Calibrates IMU, waits for “Start” keyword
2. **ROAMING**: Moves randomly until it hears “Help”
3. **SEARCHING**: Rotates and checks for nearby humans
4. **PHOTO**: Sends image to server for bounding box detection
5. **FOLLOWING**: Adjusts heading toward human using bounding box data
6. **FINISH**: If bounding box is >60% width, sends GPS and stops
7. **STOPPED**: Re-entered if “Stop” keyword is detected

## Challenges / Future Work

### Challenges

- **Microcontroller constraints**: Limited RAM/flash forced model size reductions and abandoning on-board bounding box detection
- **WiFi instability**: Arduino WiFi stack caused stack overflows, required source patching
- **Slow image transmission**: 19200-byte image → TCP bottleneck
- **Limited WiFi access**: Had to use cellular hotspot due to campus network restrictions
- **IMU inaccuracy**: Compass-based yaw angle noisy near magnets and doesn’t span full 0–360° range → Required calibration
- **IMU drift**: Caused poor long-term heading accuracy even with PID

### Future Work

- **Faster hardware**: Upgrade to microcontroller with more RAM/FLASH
- **Offline bounding box detection**: Remove dependency on server and WiFi
- **Autonomous return**: Step-retracing using stored GPS data
- **Improved localization**: Replace IMU with more accurate or fusion-based heading system

[Back](../index.html)
