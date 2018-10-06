---
layout: post
title: Android 开发 Camera和Mediarecords过程中挖坑填坑
description: Android 开发 Camera和Mediarecords过程中遇到的问题，及相关的解决方法
keywords: Android, framework
---

### 场景说明
在开发过程中需要做一个类似于远程视频的功能，就是如果没有远程观看请求，
则本地进行录像和界面显示，如果有远程观看请求则需要优先响应远程的观看。


### 具体步骤

### 遇到的问题
1. Mediarecord 没有按照(setMaxDuration)设定的最大时间来结束录制，而是直接就退出了
原因及解决方法：因为在open camera 的时候设置了preview的时候framerate和Mediarecord中
mMediaRecorder.setVideoFrameRate(this.mFrameRate)设置不一致，
最好将预览的高度和宽度也设置成相同，以避免出现分辨率不正确的问题。

