# Media Router

<p align="center"><img src="readme1.png" /></p>


## Introduction
The purpose of Media Router is to be a "mediator" between a frontend GUI Audio / Video player and a backend Multimedia Framework. It will be responsible to satisfy frontend's needs such as Open/Play/Pause/Seek/Stop functionalities but also to serve __accurate__ with the right __control flow__ and __synchronized__ the requested media frames such as Audio, Video and Subtitles.

__Accurate__ &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;: Media frames will be served at the exact timestamp that they supposed to.

__Control Flow__ &nbsp;: The incoming flow from Multimedia Framework and outgoing to GUI will be kept low (CPU/GPU/RAM).

__Synchronized__ : Ensures that all time the served media frames will be syncronized between them.

<br/>

## Design

### Layer 1 - Multimedia Framework (FFmpeg.cs)

> <a href="https://www.ffmpeg.org/">FFmpeg 4.2.2</a> library (implemented with C# bindings <a href="https://github.com/Ruslan-B/FFmpeg.AutoGen">FFmpeg.AutoGen</a> 4.2.2.2)

Demuxes the input file and configures the included media streams. It creates one thread per media stream for decoding. Additionally, it supports hardware acceleration (partially) and accurate seeking by decoding from the previous key/I frame until the requested (in case of B/P frames).

### Layer 2 - Media Router (MediaRouter.cs)

The main implementation is within the "Screamer" method that routes media frames accurately (based on frame timestamp) to the frontend. It supports Audio and Subtitles synchronization with the Video frames. Additionally, it tries to keep the frame queues low so the backend decoder will run only when required (to keep CPU/GPU/RAM low).

### Layer 3 - User Interface (UserInterface.cs)

> <a href="http://www.monogame.net/">Monogame</a> 3.7.1 & <a href="https://github.com/naudio/NAudio">NAudio</a> 1.10.0 library & <a href="https://www.codeproject.com/Tips/1193311/Csharp-Slider-Trackbar-Control-using-Windows-Forms">ColorSlider</a>

A sample GUI has been created to demonstrate Media Router's functionality. It works with both Game Engine (for taking the advantage of GPU and Game Loop) and a classic Windows Form. For subtitles will work with BOM specified, UTF-8 formats otherwise with the default system codepage (lazy support for ASS/SSA). For audio it simple runs with the NAudio library. It currently supports :- 


| Keys                  | Action                     |
| :-------------:       |:-------------:             |
| F1                    | Help / Key Bindings        |
| Drag & Drop           | Open                       |
| P / Space / Right Click| Pause / Play              |
| Left / Right Arrows   | Seeking                    |
| S                     | Stop                       |
| R                     | Keep Ratio                 |
| F                     | Full Screen / Normal Screen|
| H                     | Video Acceleration (On/Off)|
| I                     | Force Idle Mode            |
| Esc / Middle Click    | Back to Torrent File List  |
| Up / Down Arrows      | Volume Adjustment          |
| [ / ]                 | Audio Adjustment           |
| Ctrl + [ / ]          | Audio Adjustment 2         |
| ; / '                 | Subtitles Adjustment       |
| Ctrl + ; / '          | Subtitles Adjustment 2     |
| Ctrl + Up / Down      | Subtitles Location (Height)|
| Ctrl + Left / Right   | Subtitles Font Size        |

<br/>

## Changelog
#### v1.2.6 - 15/7/2020
>__Additions__

* Torrent Down Rate & Peers Stats
* UI - Removed resizing Texture while Idle/UnIdle
* New Key Bindings (Support Unicode - for any Keyboard Layout)
  * Help / Key Bindings (F1)
  * Subtitles Font Size (Ctrl + Left / Right)
  * Subtitles Location Height (Ctrl + Up / Down)
  * Subtitles Adjustment 2 - Large Step (Ctrl + ; / ')
  * Audio Adjustment 2 - Large Step (Ctrl + [ / ])

>__Issues__

* Torrent File List - Proper Sorting (AlphaNumerical)
* Play Video even when Audio fails


#### v1.2.5 - 18/6/2020
>__Issues__

* UI - Critical Issue with HW Texture Rendering (not proper resize for swapchain)
* UI - One Frame Before for VA Issue (D3D11Device Flushing requires few ms to complete)
* Audio - Broken sound until NAudio buffer would be filled
* Open - Crashing Issue Fixed
* Memory - GPU Memory Leaks Issues with HW Frames (still having some during opening - not proper free of HW Shared Textures)
* Subtitles - Font Changed, Better Synching and Rendering


#### v1.2.4 - 13/6/2020
>__Additions__

* UI / Torrent Streaming - Adding Escape / Middle Mouse Click for Torrent File List and jumping from one to another (it will continue from the point it stopped - no re-downloading)
* UI - Adding Right Mouse Click for Pause/Play
* TorSwarm - Implementing Pause/Continue Functionality (in case of different files within the torrent)

>__Issues__

* One Frame Before Issue Fixed (by flushing the FFmpeg's D3D11Device)
* Full Screen Aspect Ratio Issue Fixed (in cases display.Width / aspectRatio > display.Height)
* Linesize for Software Texture Fixed (adding also latest FFmpeg's swscale now)
* Crashing Issues during Opening (decoder.Stop was seeking at 0 / replaced it with Close - Pause)
* After seeking was always keep playing on Torrent Streaming (fixed by remembering previous status)
* TorSwarm - Fixed an Issue with Include/Exlude Functionality (it was deleting the previous status of excluded files)


#### v1.2.3 - 9/6/2020
>__Additions__

* UI / Torrent Streaming - Auto Select First Media File (if no other exists)
* Better Handling Multi-Monitors
* FFmpeg.AutoGen Libraries Update to latest

>__Issues__

* Seeking on Torrent Streaming was crashing

#### v1.2.2 - 3/6/2020
>__Additions__

* Implementing (with SharpDX) Direct3D 11 Video Decoding & Acceleration Support (NV12 Pixel Formats)
* Better Image Quality & CPU/RAM Performace (Video Frames lifecycle only within the GPU)
* Faster Torrent Streaming (By New Request Piece Algorithm & Disabling Embedded Subtitles)
* 'H' Key for Video Acceleration (On/Off) [Requires re-opening the input]
* FFmpeg Libraries Update to latest (except swscale)

>__Issues__

* Subtitles Issues
* Performance Issues with the UI

#### v1.2 - 23/5/2020
>__Additions__

* Torrent files / Magnet links (Drag & Drop) for torrent streaming (by merging with my other project  <a href="https://github.com/SuRGeoNix/TorSwarm">TorSwarm</a>) and creating the new __MediaStreamer__ class for general use later on
* Re-design the main Screamer's implementation and Syncing, using seperate threads (Audio/Video/Subs) Screamers and screaming each stream's frame at exact timestamp of each one
* Support for more media formats (the most common, still will not handle correctly ts/vob etc)
* Fast seeking with better FFmpeg implementation and threading so it will not hang the UI
* Audio stereo (2 channels) output for any input and sample rate (it was supporting only 1 channel and 48Khz rate)
* NAudio package updated to 1.10, FFmpeg can be updated to latest (for low quality, use the previous swscale-5.dll)

>__Issues__
* Audio/Subtitles seeking issues (it was going though the whole file) on matroska formats (avformat_seek_file can't seek with other stream but the video stream)
* NAudio was running at the main UI thread and it was hanging
* Performance issue fixed that was hanging the UI (high cpu/gpu) and was breaking the audio and syncing
* Changed Queues to ConcurrentQueues as they had issues with threading 

#### v1.1 - 7/11/2019 (First Release)
<br/>

## Remarks
I have worked on this project for education, fun and programming exercise and I've made it available to the public in case you will find it useful for similar reasons. It's always fun as programmers to have our own media player and play around. Any suggestions are always welcome!

<p align="center"><img src="readme2.png" /></p>