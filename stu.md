[![WebRTC multiparty video conference](https://testrtc.com/wp-content/uploads/2016/03/201603-Multiparty-video.jpg)](https://testrtc.com/different-multiparty-video-conferencing/)

[MCU](https://webrtcglossary.com/mcu/), [SFU](http://webrtcglossary.com/sfu), [Mesh](https://webrtcglossary.com/mesh/) – what do they really mean? We decided to take all these techniques to a spin to see what goes on on the network.

To that end, we used some simple test scripts in testRTC and handpicked a service that uses each of these techniques:

- For mesh we used [appear.in](https://appear.in/)
- For SFU we used [Talky](https://talky.io/)
- For MCU we used [Blue Jeans](http://bluejeans.com/)

We used 4 browsers for each test. All running Chrome 48 (the current stable version). All from the same data center. All using the same 720p video stream as their camera source.

While the test lengths varied across tests, we will be interested to see the average bitrate expenditure of each to understand the differences.

### Mesh

appear.in runs a mesh call. It means that each user will need to send its media to all other users in the session – as well as receive all the media streams from them.

This is how it looks like:

![mesh video architecture](https://testrtc.com/wp-content/uploads/2016/03/201603-mesh.png)

I’ve opened up an ad-hoc room there and got 4 of our browser agents into it. Waited about a minute and collected the results:

![appear.in mesh video](https://testrtc.com/wp-content/uploads/2016/03/201603-mesh-1.png)

Nothing much to see here. Incoming and outgoing video across the whole test is rather similar, if somewhat high.

Looking at one of the browser’s media channels tells the story:

![appear.in mesh video](https://testrtc.com/wp-content/uploads/2016/03/201603-mesh-2.png)

This agent has 3 outgoing and 3 incoming voice and video channels.

Average bitrate on the video channel is around 1.2 mbps, which means our agent runs about 3.6 megabytes uplink and downlink. Not trivial.

### **SFU**

**Talky uses Jitsi for its SFU implementation. It means that it doesn’t process video but rather routes it to everyone who needs it. Each browser sends its media to the SFU, which then forwards that media to all other participants.**

**This is how it looks like:**

**![sfu video architecture](https://testrtc.com/wp-content/uploads/2016/03/201603-sfu.png)**

**I took 4 browsers in testRTC and pointed them at a single Talky session. Here’s what the report showed:**

**![Talky SFU video](https://testrtc.com/wp-content/uploads/2016/03/201603-sfu-1.png)**

**The main thing to not there is that in total, the browsers we used processed a lot more incoming media than outgoing one (at a rate of 3 to 1). This shouldn’t surprise us. Look at how one of these browsers reports its media channels:**

**![Talky SFU video](https://testrtc.com/wp-content/uploads/2016/03/201603-sfu-2.png)**

**1 outgoing audio and video channel and then 3 incoming audio and video channels. There’s another empty video channel – Talky is probably using that for incoming screen sharing.**

**Note how in this case the same machines with the same network performance did a lot better. The outgoing video channel gets to almost 2.5 mbps bitrate. Almost twice as much as the mesh was capable of using. To make it clear – mesh doesn’t scale well.**

### MCU

For an MCU I picked BlueJeans service. We’ve been playing with it a bit on a demo account so I took the time to take a quick capture of a session. Being architectured around an MCU means that each browser sends a single video stream. The MCU takes all these video streams and composes them into a single video stream that is then sent to each participant separately.

![mcu video architecture](https://testrtc.com/wp-content/uploads/2016/03/201603-mcu.png)

As with the other two experiments, I used 4 browsers with this MCU, receiving this report highlights:

![BlueJeans MCU video](https://testrtc.com/wp-content/uploads/2016/03/201603-mcu-1.png)

Total kilobits here is rather similar. It seems that in total, browsers received less than they sent out.

Drilling down into a single browser report, we see the following channels:

![BlueJeans MCU video](https://testrtc.com/wp-content/uploads/2016/03/201603-mcu-2.png)

Single incoming and a single outgoing audio and video channels. We have an additional incoming/outgoing video channel with no data on it – probably saved for screen sharing. While similar to how Talky does it, BlueJeans opens up an extra outgoing channel by default while Talky doesn’t.

Outgoing bitrate averages at 1.2 mbps – a lot lower than the 2.5 mbps in Talky. I assume that’s because BlueJeans limited the bitrate from the browser, which actually makes a lot of sense for 720p video stream. The incoming video is even lower at 455 kbps bitrate on average.

This didn’t make sense to me, so I dug a bit deeper into some of our video charts and found this:

![BlueJeans MCU video](https://testrtc.com/wp-content/uploads/2016/03/201603-mcu-3.png)

So BlueJeans successfully managers to get its outgoing video from the MCU towards the browser up to the same 1.2 mbps bitrate. Thinking about it, I shouldn’t be surprised. Talky and appear.in are ad-hoc services, while BlueJeans is a full service with business logic in it – getting all browsers into the session takes more time with it, especially with how we’ve written the script for it. We have a full minute here from the browser showing its local video until it really “connects” to the conference.

Another interesting tidbit is that Chrome gets its bitrate to 1.2 quite fast – something Google took care of in 2015. BlueJeans takes a slower route towards that 1.2mbps taking about half a minute to get there.

### So What?

Video comes in different shapes and sizes.

WebRTC reduces a lot of the decisions we had to make and takes care of most browser related media issues, but it is quite flexible – different services use it differently to get to the same use case – here multiparty video chat.

If you are looking to understand your WebRTC service better and at the same time automate your testing and monitoring – [try out testRTC](https://app.testrtc.com/).