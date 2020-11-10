# Getting started with media devices

When developing for the web, the WebRTC standard provides APIs for accessing cameras and microphones connected to the computer or smartphone. These devices are commonly referred to as Media Devices and can be accessed with JavaScript through the `navigator.mediaDevices` object, which implements the `MediaDevices` interface. From this object we can enumerate all connected devices, listen for device changes (when a device is connected or disconnected), and open a device to retrieve a Media Stream (see below).

The most common way this is used is through the function `getUserMedia()`, which returns a promise that will resolve to a `MediaStream` for the matching media devices. This function takes a single `MediaStreamConstraints` object that specifies the requirements that we have. For instance, to simply open the default microphone and camera, we would do the following.

[USING PROMISES](https://webrtc.org/getting-started/media-devices#using-promises)[USING ASYNC/AWAIT](https://webrtc.org/getting-started/media-devices#using-asyncawait)

```js
const constraints = {
    'video': true,
    'audio': true
}
navigator.mediaDevices.getUserMedia(constraints)
    .then(stream => {
        console.log('Got MediaStream:', stream);
    })
    .catch(error => {
        console.error('Error accessing media devices.', error);
    });
```

The call to `getUserMedia()` will trigger a permissions request. If the user accepts the permission, the promise is resolved with a `MediaStream` containing one video and one audio track. If the permission is denied, a `PermissionDeniedError` is thrown. In case there are no matching devices connected, a `NotFoundError` will be thrown.

The full API reference for the `MediaDevices` interface is available at [MDN web docs](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices).

## Querying media devices

In a more complex application, we will most likely want to check all the connected cameras and microphones and provide the appropriate feedback to the user. This can be done by calling the function `enumerateDevices()`. This will return a promise that resolves to an array of `MediaDevicesInfo` that describe each known media device. We can use this to present a UI to the user which let's them pick the one they prefer. Each `MediaDevicesInfo` contains a property named `kind` with the value `audioinput`, `audiooutput` or `videoinput`, indicating what type of media device it is.

[USING PROMISES](https://webrtc.org/getting-started/media-devices#using-promises)[USING ASYNC/AWAIT](https://webrtc.org/getting-started/media-devices#using-asyncawait)

```js
function getConnectedDevices(type, callback) {
    navigator.mediaDevices.enumerateDevices()
        .then(devices => {
            const filtered = devices.filter(device => device.kind === type);
            callback(filtered);
        });
}

getConnectedDevices('videoinput', cameras => console.log('Cameras found', cameras));
```

## Listening for devices changes

Most computers support plugging in various devices during runtime. It could be a webcam connected by USB, a Bluetooth headset, or a set of external speakers. In order to properly support this, a web application should listen for the changes of media devices. This can be done by adding a listener to `navigator.mediaDevices` for the `devicechange` event.

```js
// Updates the select element with the provided set of cameras
function updateCameraList(cameras) {
    const listElement = document.querySelector('select#availableCameras');
    listElement.innerHTML = '';
    cameras.map(camera => {
        const cameraOption = document.createElement('option');
        cameraOption.label = camera.label;
        cameraOption.value = camera.deviceId;
    }).forEach(cameraOption => listElement.add(cameraOption));
}

// Fetch an array of devices of a certain type
async function getConnectedDevices(type) {
    const devices = await navigator.mediaDevices.enumerateDevices();
    return devices.filter(device => device.kind === type)
}

// Get the initial set of cameras connected
const videoCameras = getConnectedDevices('videoinput');
updateCameraList(videoCameras);

// Listen for changes to media devices and update the list accordingly
navigator.mediaDevices.addEventListener('devicechange', event => {
    const newCameraList = getConnectedDevices('video');
    updateCameraList(newCameraList);
});
```

## Media constraints

The constraints object, which must implement the `MediaStreamConstraints` interface, that we pass as a parameter to `getUserMedia()` allows us to open a media device that matches a certain requirement. This requirement can be very loosely defined (audio and/or video), or very specific (minimum camera resolution or an exact device ID). It is recommended that applications that use the `getUserMedia()` API first check the existing devices and then specifies a constraint that matches the exact device using the `deviceId` constraint. Devices will also, if possible, be configured according to the constraints. We can enable echo cancellation on microphones or set a specific or minimum width and height of the video from the camera.

```js
async function getConnectedDevices(type) {
    const devices = await navigator.mediaDevices.enumerateDevices();
    return devices.filter(device => device.kind === type)
}

// Open camera with at least minWidth and minHeight capabilities
async function openCamera(cameraId, minWidth, minHeight) {
    const constraints = {
        'audio': {'echoCancellation': true},
        'video': {
            'deviceId': cameraId,
            'width': {'min': minWidth},
            'height': {'min': minHeight}
            }
        }

    return await navigator.mediaDevices.getUserMedia(constraints);
}

const cameras = getConnectedDevices('videoinput');
if (cameras && cameras.length > 0) {
    // Open first available video camera with a resolution of 1280x720 pixels
    const stream = openCamera(cameras[0].deviceId, 1280, 720);
}
```

The full documentation for the `MediaStreamConstraints` interface can be found on the [MDN web docs](https://developer.mozilla.org/en-US/docs/Web/API/MediaStreamConstraints).

## Local playback

Once a media device has been opened and we have a `MediaStream` available, we can assign it to a video or audio element to play the stream locally.

```js
async function playVideoFromCamera() {
    try {
        const constraints = {'video': true, 'audio': true};
        const stream = await navigator.mediaDevices.getUserMedia(constraints);
        const videoElement = document.querySelector('video#localVideo');
        videoElement.srcObject = stream;
    } catch(error) {
        console.error('Error opening video camera.', error);
    }
}
```

The HTML needed for a typical video element used with `getUserMedia()` will usually have the attributes `autoplay` and `playsinline`. The `autoplay` attribute will cause new streams assigned to the element to play automatically. The `playsinline` attribute allows video to play inline, instead of only in full screen, on certain mobile browsers. It is also recommended to use `controls="false"` for live streams, unless the user should be able to pause them.

```html
<html>
<head><title>Local video playback</video></head>
<body>
    <video id="localVideo" autoplay playsinline controls="false"/>
</body>
</html>
```

Was this page helpful?

Except as otherwise noted, the content of this page is licensed under the [Creative Commons Attribution 4.0 License](https://creativecommons.org/licenses/by/4.0/), and code samples are licensed under the [Apache 2.0 License](https://www.apache.org/licenses/LICENSE-2.0). For details, see the [Google Developers Site Policies](https://developers.google.com/site-policies). Java is a registered trademark of Oracle and/or its affiliates.

Last updated 2020-07-30 UTC.



Key topics[WebRTC on MDN](https://developer.mozilla.org/en-US/docs/Glossary/WebRTC)