# YouTube Stream Url Format
There has been some question about how to convert the stream url format from YouTube to work with the Panasonic AG-CX10. We will step through the process to clear up any confusion. 

TLDR; The format for the URL is:

rtmp://a.rtmp.youtube.com/live2/{StreamKey}

So a complete sample might look like this:

rtmp://a.rtmp.youtube.com/live2/stzz-7q1s-8tr9-1geb-92wp

### Video Walkthrough
If you prefer, you can watch the video walkthrough:
<iframe width="560" height="315" class="video-frame" src="https://www.youtube.com/embed/Yfs0FzpPDBc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Steps
1. First navigate to the YouTube and login.
![Image 1](https://raw.githubusercontent.com/mobiletonster/blogposts/main/video/streaming/images/Image1.jpg#screenshot "youtube main screen")

2. Launch YouTube Studio from the menu.
![Image 2](https://raw.githubusercontent.com/mobiletonster/blogposts/main/video/streaming/images/Image2.jpg#screenshot "youtube main screen + menu")

3. YouTube Studio Dashboard will load.
![Image 3](https://raw.githubusercontent.com/mobiletonster/blogposts/main/video/streaming/images/Image3.jpg#screenshot "youtube dashboard")

4. Click the GoLive Button to navigate to Streaming Panel.
![Image 4](https://raw.githubusercontent.com/mobiletonster/blogposts/main/video/streaming/images/Image4.jpg#screenshot "youtube streaming panel")
In the lower left corner, locate the Stream URL and the Stream Key

5. Paste the Stream URL portion into notepad
![Image 5](https://raw.githubusercontent.com/mobiletonster/blogposts/main/video/streaming/images/Image5.jpg#screenshot "url in notepad")

6. add a `/` slash and then paste the stream key after
![Image 6](https://raw.githubusercontent.com/mobiletonster/blogposts/main/video/streaming/images/Image6.jpg#screenshot "url + key in notepad")

7. Copy combined url + key into P2NetGen software RTMP tab
![Image 7](https://raw.githubusercontent.com/mobiletonster/blogposts/main/video/streaming/images/Image7.jpg#screenshot "url + key in notepad")
Then export to an SD Card and follow the steps to import from card to the camera.

And those are the steps and the format for the stream url + key.

