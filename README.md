# Arduino Continuous Over-The-Air updates

How far can Continuous Integration/Continuous Delivery (CI/CD) and over-the-air (OTA) updates be applied to a simple Arduino sketch?


## 0) Start with the most basic sketch - Blink

Nothing fancy here, just the well-known sketch to blink the built-in LED.<br>
(source: https://github.com/arduino/arduino-examples/blob/main/examples/01.Basics/Blink/Blink.ino)


## 1) Continuous Integration: use GitHub Actions to compile the sketch

This uses standard GitHub Actions and https://github.com/arduino/compile-sketches to compile the sketch.


## 2) Continuous Delivery: create a release for every pushed tag

Here things are getting interesting. Automating the release process itself is straightforward. I picked https://github.com/softprops/action-gh-release. Other ready-to-use GitHub Actions are available, too.<br>
However, since later on we want to automate also OTA updates, there are 2 common caveats to be considered:

### Caveat 1: The Arduino needs to know where to check for new firmware versions.

GitHub release URLs are difficult to predict because they include both the tag name and the asset name, and naming schemes may vary. GitHub provides a few static URLs to obtain info related to the latest release:
- `https://github.com/<OWNER>/<REPO>/releases/latest`<br>
  The webpage for the latest release. Rather large and complex to parse.
- `https://api.github.com/repos/<OWNER>/<REPO>/releases/latest`<br>
  JSON metadata describing the latest release. Better than the webpage, but still contains far more information than what is actually needed.
- `https://github.com/<OWNER>/<REPO>/releases/latest/download/<ASSET_NAME>`<br>
   A direct link to a single asset in the latest release. Can be easily used if <ASSET_NAME> is static (i.e. it contains no version or build numbers).

### Caveat 2: Since the static URL `https://github.com/<OWNER>/<REPO>/releases/latest/download/<STATIC_ASSET_NAME>` contains no information regarding the version or build date, how does the Arduino determine if it needs to download and install the firmware?

You don't want to periodically download and flash the (eventually updated) firmware if there is no need to do so.

### Solution

1. Create a custom JSON metadata file with a fixed name and publish it as an additional release asset. This way, you have a static URL always pointing to the latest version of the metadata file. The file is very small, it contains only:
   - A unique build number
   - The download URL for the corresponding HEX file

2. Inject the same unique build number into the sketch during compilation in CI.

3. Publish the metadata and the firmware files together as release assets. The naming conventions for tags and firmware files are irrelevant, as long as the correct URL to the firmware file is included in the metadata file.

4. The Arduino periodically retrieves the lightweight metadata file and compares the build number it contains with the build number embedded in its currently running firmware.
   - If the metadata contains a higher build number, download and install the firmware.
   - Otherwise, wait and check again after one minute.

### Notes

Keeping the metadata as small as possible is important, since this file will be frequently downloaded to and parsed on a embedded device with limited RAM and CPU capabilities.

| URL                                                                           | Type | Size     | Suitability |
|-------------------------------------------------------------------------------|------|----------|-------------|
| https://github.com/mast-eu/ArduinoCOTA/releases/latest                        | HTML | > 200 kB | 🟠 Larger than the RAM of an Arduino. |
| https://api.github.com/repos/mast-eu/ArduinoCOTA/releases/latest              | JSON | ~ 3.5 kB | 🟡 Fits into the RAM of _newer_ Arduinos, but can be further reduced. |
| https://github.com/mast-eu/ArduinoCOTA/releases/latest/download/metadata.json | JSON | < 0.2 kB | 🟢 Small and easy to parse. |
