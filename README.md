# BrokenEye

> [!WARNING]
> This software is free and does not attempt to make a profit. \
> It is intended for non-commercial use only. \
> The software is provided "as is", without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose and noninfringement. In no event shall the authors or copyright holders be liable for any claim, damages or other liability, whether in an action of contract, tort or otherwise, arising from, out of or in connection with the software or the use or other dealings in the software.

BrokenEye is an application that will allow you to obtain more detailed data from the Tobii Eye Tracker, namely:

- Gaze direction
- Eye convergence
- Pupil diameter
- Blink / Eye openness ([more details below](#using-an-external-application-to-obtain-eye-openness-data))
- Streaming images from cameras!

It also allows you to:
- Save and load calibration data
- Record and playback data from device (this is a regular zip).

## Supported devices

- Pimax Crystal
- HP Reverb G2 Omnicept Edition

If your headset also supports Tobii tracking and you would like to add support for it to BrokenEye, please open an issue:
- indicate the full name of the headset
- check the functionality in [VRCFaceTracking](https://docs.vrcft.io/docs/vrcft-software/vrcft#install-vrcfacetracking) together with the [PimaxCrystalAdvanced](https://github.com/ghostiam/PimaxCrystalAdvanced) plugin (without BrokenEye).

If the plugin works, then there is a high chance of working with BrokenEye.

## BrokenEye features video:
[![Pimax Crystal with improved eye tracking using additional software](https://img.youtube.com/vi/a_0a2vSXWP0/0.jpg)](https://www.youtube.com/watch?v=a_0a2vSXWP0 "Pimax Crystal with improved eye tracking using additional software")

## How to install

1. Download the latest version of the application
   from [releases](https://github.com/ghostiam/BrokenEye/releases/latest).
2. Just launch the application.

## How to use

After launching the application you can choose:

![image](_assets/image.png)

1) Which device will it connect to?
2) Open data you have already recorded (or record if you are already connected to the device)
3) Do not forget to enable or disable preview of data and images from cameras,
   whenever you need it, since previewing will create unnecessary load on your computer.
4) Save or load Eye Tracker calibration data (be sure to calibrate the device before saving!).
5) Server settings for streaming data and images from the camera.
6) OSC server settings to receive eye openness
   data [from another application](#using-an-external-application-to-obtain-eye-openness-data).
7) OSC server parameters.

All settings are saved automatically.

> [!NOTE]
> You don't need to enable recording to use the application! \
> Recording is only necessary if you want to play back the data you received from the device in the future,
> for example, to configure an external application so as not to wear a headset.

## Sending bug reports

If you find a bug, please report it in the [issues](https://github.com/ghostiam/BrokenEye/issues).
Please attach a logs and configuration file to the bug report, which can be found in the `C:\Users\<USER>\AppData\Roaming\fyne\com.ghostiam.BrokenEye` folder.

## Use in VRChat

In order to use in VRChat, you should
install [VRCFaceTracking](https://docs.vrcft.io/docs/vrcft-software/vrcft#install-vrcfacetracking)
and a plugin for it [PimaxCrystalAdvanced](https://github.com/ghostiam/PimaxCrystalAdvanced)

First run BrokenEye and then VRCFaceTracking.

## Using an external application to obtain eye openness data

Since we can only receive data on blinking from the tracker, to obtain data on eye openness, it is necessary
use another application that will send data to BrokenEye via the OSC protocol,
for example [EyeTrackVR](https://github.com/EyeTrackVR/EyeTrackVR)

[How to configure EyeTrackVR](ETVR-configure.md)

## How to get data from BrokenEye

There are 2 APIs for receiving data:

- [HTTP](#http-api) (only for camera images)
- [RAW](#raw-api) - data is transmitted over a TCP connection, in JSON format for processed data, and images in raw
  format.

### HTTP API

Images from cameras are transmitted as [MJPEG](https://en.wikipedia.org/wiki/Motion_JPEG) stream, which are available
via
address:

For the left eye:

```
http://127.0.0.1:5555/eye/left
```

For the right eye:

```
http://127.0.0.1:5555/eye/right
```

You can also preview images in your browser by going to:

```
http://127.0.0.1:5555/
```

The default port is `5555`, but it can be changed in the application.

### RAW API

To receive data in RAW format, you need to connect via TCP to the application on port `5555`.

The request for data looks like this:

|   ID   | description                                  |
|:------:|:---------------------------------------------|
| `0x00` | Request for eye tracking data in JSON format |
| `0x01` | Request for raw image of left eye            |
| `0x02` | Request for raw image of right eye           |

we send:

|  ID  |
|:----:|
| byte |

we get in the loop:

|  ID  |          Size           |      Data      |
|:----:|:-----------------------:|:--------------:|
| byte | 4 bytes (little endian) | Array of bytes |

An example in C# for obtaining eye tracking data can be viewed
in [this](https://github.com/ghostiam/PimaxCrystalAdvanced/blob/main/BrokenEye/Client.cs) file.

#### Eye tracking data format:

<details>
<summary>JSON example</summary>

```json5
{
  "left": {
    "gaze_direction_is_valid": false,
    "gaze_direction": [
      // X
      0,
      // Y
      0,
      // Z
      0
    ],
    "pupil_diameter_is_valid": false,
    "pupil_diameter_mm": -1,
    "pupil_position_on_image_is_valid": false,
    "pupil_position_on_image": [
      // X
      -1,
      // Y
      -1
    ],
    "openness_is_valid": true,
    "openness": 1
  },
  "right": {
    "gaze_direction_is_valid": false,
    "gaze_direction": [
      // X
      0,
      // Y
      0,
      // Z
      0
    ],
    "pupil_diameter_is_valid": false,
    "pupil_diameter_mm": -1,
    "pupil_position_on_image_is_valid": false,
    "pupil_position_on_image": [
      // X
      -1,
      // Y
      -1
    ],
    "openness_is_valid": true,
    "openness": 1
  }
}
```

</details>

#### Raw image format:

|  Width  | Height  | Bit per pixel | Raw data |
|:-------:|:-------:|:-------------:|:--------:|
| 4 bytes | 4 bytes |    4 bytes    | N bytes  |

Where N is the size of the image in bytes:
```
N = Width * Height * (Bit per pixel / 8)
```
