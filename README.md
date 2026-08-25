# Arduino Continuous Over-The-Air updates

How far can Continuous Integration/Continuous Delivery (CI/CD) and over-the-air (OTA) updates be applied to a simple Arduino sketch?


## 0) Start with the most basic sketch - Blink

Nothing fancy here, just the well-known sketch to blink the built-in LED.<br>
(source: https://github.com/arduino/arduino-examples/blob/main/examples/01.Basics/Blink/Blink.ino)


## 1) Continuous Integration: use GitHub Actions to compile the sketch

This uses standard GitHub Actions and https://github.com/arduino/compile-sketches to compile the sketch.
