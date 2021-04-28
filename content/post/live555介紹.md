---
title: "live555介紹"
date: 2021-03-05T15:14:18+08:00
tags : [ "live555"]
layout: post
type:  "post"
draft: false
---

這次是介紹偏向live555，但是live555實在是一個很大的專案，
所以我也只是看了其中一小部分。


這邊先介紹live555處理h264 video的部分：

```objc
-(void)HandleForH264Video:(const uint8_t *)frameData frameDataLength:(int)frameDataLength presentationTime:(struct timeval)presentationTime durationInMicroseconds:(unsigned int)duration
```
傳入data，顯示時間，顯示多久。

```objc
do {

......

} while (0);
```

這個迴圈的用途是用break時很好用，可以繼續往下執行。

```objc
        int iType = frameData[0] & 0x1f;
        
        if(iType == 0X1 && frameData[1] == 0x88)//if nal type is non-idr but frame slice is i or si slice
            iType = 0x5;
        
        VideoFrame *tmpVideoFrame=nil;

        if(iType == 0X7)//SPS
        {
            if(m_SPSFrame.m_pRawData != NULL) // just need once
                break;
            
            tmpType = @"SPS";
            
            m_SPSFrame.m_intFrameType = SPS_FRAME;
            m_SPSFrame.m_uintFrameLenth = frameDataLength + sizeof(m_aryH264StartCode);
            m_SPSFrame.m_pRawData = (unsigned char*)malloc(sizeof(m_aryH264StartCode) + frameDataLength);
            memcpy(m_SPSFrame.m_pRawData, m_aryH264StartCode, sizeof(m_aryH264StartCode));
            memcpy(m_SPSFrame.m_pRawData + sizeof(m_aryH264StartCode), frameData, frameDataLength);
            [self.m_VideoDecoder setSPSFrame:m_SPSFrame];
            break;
        }
        else if(iType == 0X8)//pps
        {
            if(m_PPSFrame.m_pRawData != NULL) // just need once
                break;
            tmpType = @"PPS";
            
            m_PPSFrame.m_intFrameType = SPS_FRAME;
            m_PPSFrame.m_uintFrameLenth = frameDataLength + sizeof(m_aryH264StartCode);
            m_PPSFrame.m_pRawData = (unsigned char*)malloc(sizeof(m_aryH264StartCode) + frameDataLength);
            memcpy(m_PPSFrame.m_pRawData, m_aryH264StartCode, sizeof(m_aryH264StartCode));
            memcpy(m_PPSFrame.m_pRawData + sizeof(m_aryH264StartCode), frameData, frameDataLength);
            [self.m_VideoDecoder setPPSFrame:m_PPSFrame];
            break;
        }
```

int iType = frameData[0] & 0x1f; //取出一個值判斷nalu type，

 if(iType == 0X7)  就是SPS，進行以下處理：
            if(m_SPSFrame.m_pRawData != NULL) // just need once
                break;
```objc
            tmpType = @"SPS";
            
            m_SPSFrame.m_intFrameType = SPS_FRAME;
            m_SPSFrame.m_uintFrameLenth = frameDataLength + sizeof(m_aryH264StartCode);
            m_SPSFrame.m_pRawData = (unsigned char*)malloc(sizeof(m_aryH264StartCode) + frameDataLength);
            memcpy(m_SPSFrame.m_pRawData, m_aryH264StartCode, sizeof(m_aryH264StartCode));
            memcpy(m_SPSFrame.m_pRawData + sizeof(m_aryH264StartCode), frameData, frameDataLength);
            [self.m_VideoDecoder setSPSFrame:m_SPSFrame];  // For VideoToolBox(NV12)
```

else if(iType == 0X8) 就是PPS，進行跟SPS一樣的處理。

```objc
        else if(iType == 0X5)//I-Frame
        {
            if(m_PPSFrame.m_pRawData == NULL || m_SPSFrame.m_pRawData == NULL)
                break;
            
            tmpVideoFrame = [[VideoFrame alloc] init];
            tmpType = @"I-frame";
            pCount = 0;
            
            //For YUV420, need to insert the sps and pps into begin of the video frame.
            //                if(!self.m_blnReceiveFirstIFrame){
            //                    int iPreheaderTotal = (int)(m_SPSFrame.m_uintFrameLenth + m_PPSFrame.m_uintFrameLenth);
            //
            //                    tmpVideoFrame.m_pRawData = (uint8_t*)malloc(frameDataLength + sizeof(m_aryH264StartCode) + iPreheaderTotal );
            //                    tmpVideoFrame.m_uintFrameLenth = frameDataLength + sizeof(m_aryH264StartCode) + iPreheaderTotal  ;
            //                    tmpVideoFrame.m_intFrameSEQ = pCount;
            //                    tmpVideoFrame.m_intFrameType = VIDEO_I_FRAME;
            //                    memcpy(tmpVideoFrame.m_pRawData, m_SPSFrame.m_pRawData, (int)m_SPSFrame.m_uintFrameLenth);
            //                    memcpy((tmpVideoFrame.m_pRawData + (int)m_SPSFrame.m_uintFrameLenth), m_PPSFrame.m_pRawData, (int)m_PPSFrame.m_uintFrameLenth);
            //                    memcpy((tmpVideoFrame.m_pRawData + iPreheaderTotal), m_aryH264StartCode, sizeof(m_aryH264StartCode));
            //                    memcpy(tmpVideoFrame.m_pRawData + iPreheaderTotal + sizeof(m_aryH264StartCode) , frameData, frameDataLength);
            //                }
            //                else{
            //                    int iPreheaderTotal = 0;
            //
            //                    tmpVideoFrame.m_pRawData = (uint8_t*)malloc(frameDataLength + sizeof(m_aryH264StartCode) + iPreheaderTotal );
            //                    tmpVideoFrame.m_uintFrameLenth = frameDataLength + sizeof(m_aryH264StartCode) + iPreheaderTotal  ;
            //                    tmpVideoFrame.m_intFrameSEQ = pCount;
            //                    tmpVideoFrame.m_intFrameType = VIDEO_I_FRAME;
            //                    memcpy((tmpVideoFrame.m_pRawData + iPreheaderTotal), m_aryH264StartCode, sizeof(m_aryH264StartCode));
            //                    memcpy(tmpVideoFrame.m_pRawData + iPreheaderTotal + sizeof(m_aryH264StartCode) , frameData, frameDataLength);
            //                }
            
            
            //For VideoToolBox(NV12), not need to insert the sps and pps into begin of the video frame.
            int iPreheaderTotal = 0;
            
            tmpVideoFrame.m_pRawData = (uint8_t*)malloc(frameDataLength + sizeof(m_aryH264StartCode) + iPreheaderTotal );
            tmpVideoFrame.m_uintFrameLenth = frameDataLength + sizeof(m_aryH264StartCode) + iPreheaderTotal  ;
            tmpVideoFrame.m_intFrameSEQ = pCount;
            tmpVideoFrame.m_intFrameType = VIDEO_I_FRAME;
            memcpy((tmpVideoFrame.m_pRawData + iPreheaderTotal), m_aryH264StartCode, sizeof(m_aryH264StartCode));
            memcpy(tmpVideoFrame.m_pRawData + iPreheaderTotal + sizeof(m_aryH264StartCode) , frameData, frameDataLength);
            
            if(!self.m_blnReceiveFirstIFrame)
                [self.eventDelegate connectSuccess:self];
            
            self.m_blnReceiveFirstIFrame = YES;
            
        }
```

else if(iType == 0X5)//I-Frame 進行以下處理：
```objc
            if(m_PPSFrame.m_pRawData == NULL || m_SPSFrame.m_pRawData == NULL)
                break;
```
一定要有SPS跟PPS。
            
中間註解起來的那段是一開始用ffmpeg decode的，需要先將SPS跟PPS加入到前頭。
然後用memcpy 拷貝到剛才建立的tmpVideoFrame中，注意，這裏的memcpy不能少，
因為每一次live555收到新的frame時data都會覆蓋掉，所以一定要拷貝出來。
```objc
            //For YUV420, need to insert the sps and pps into begin of the video frame.
            //                if(!self.m_blnReceiveFirstIFrame){
            //                    int iPreheaderTotal = (int)(m_SPSFrame.m_uintFrameLenth + m_PPSFrame.m_uintFrameLenth);
            //
            //                    tmpVideoFrame.m_pRawData = (uint8_t*)malloc(frameDataLength + sizeof(m_aryH264StartCode) + iPreheaderTotal );
            //                    tmpVideoFrame.m_uintFrameLenth = frameDataLength + sizeof(m_aryH264StartCode) + iPreheaderTotal  ;
            //                    tmpVideoFrame.m_intFrameSEQ = pCount;
            //                    tmpVideoFrame.m_intFrameType = VIDEO_I_FRAME;
            //                    memcpy(tmpVideoFrame.m_pRawData, m_SPSFrame.m_pRawData, (int)m_SPSFrame.m_uintFrameLenth);
            //                    memcpy((tmpVideoFrame.m_pRawData + (int)m_SPSFrame.m_uintFrameLenth), m_PPSFrame.m_pRawData, (int)m_PPSFrame.m_uintFrameLenth);
            //                    memcpy((tmpVideoFrame.m_pRawData + iPreheaderTotal), m_aryH264StartCode, sizeof(m_aryH264StartCode));
            //                    memcpy(tmpVideoFrame.m_pRawData + iPreheaderTotal + sizeof(m_aryH264StartCode) , frameData, frameDataLength);
            //                }
            //                else{
            //                    int iPreheaderTotal = 0;
            //
            //                    tmpVideoFrame.m_pRawData = (uint8_t*)malloc(frameDataLength + sizeof(m_aryH264StartCode) + iPreheaderTotal );
            //                    tmpVideoFrame.m_uintFrameLenth = frameDataLength + sizeof(m_aryH264StartCode) + iPreheaderTotal  ;
            //                    tmpVideoFrame.m_intFrameSEQ = pCount;
            //                    tmpVideoFrame.m_intFrameType = VIDEO_I_FRAME;
            //                    memcpy((tmpVideoFrame.m_pRawData + iPreheaderTotal), m_aryH264StartCode, sizeof(m_aryH264StartCode));
            //                    memcpy(tmpVideoFrame.m_pRawData + iPreheaderTotal + sizeof(m_aryH264StartCode) , frameData, frameDataLength);
            //                }
```            
      

而改成用 video tool box decode後，不要將SPS跟PPS加入，而是copy start code將frame data就好。

 ```objc     
            //For VideoToolBox(NV12), not need to insert the sps and pps into begin of the video frame.
            int iPreheaderTotal = 0;
            
            tmpVideoFrame.m_pRawData = (uint8_t*)malloc(frameDataLength + sizeof(m_aryH264StartCode) + iPreheaderTotal );
            tmpVideoFrame.m_uintFrameLenth = frameDataLength + sizeof(m_aryH264StartCode) + iPreheaderTotal  ;
            tmpVideoFrame.m_intFrameSEQ = pCount;
            tmpVideoFrame.m_intFrameType = VIDEO_I_FRAME;
            memcpy((tmpVideoFrame.m_pRawData + iPreheaderTotal), m_aryH264StartCode, sizeof(m_aryH264StartCode));
            memcpy(tmpVideoFrame.m_pRawData + iPreheaderTotal + sizeof(m_aryH264StartCode) , frameData, frameDataLength);
            



        else if(iType == 0X1)//P-Frame
        {  
            pCount++;
            
            tmpVideoFrame = [[VideoFrame alloc] init];
            tmpType = @"P-frame";
            
            if (!m_blnReceiveFirstIFrame)
            {
                break;
            }
            
            tmpVideoFrame.m_intFrameSEQ = pCount ;
            tmpVideoFrame.m_intFrameType = VIDEO_P_FRAME;
            tmpVideoFrame.m_uintFrameLenth = sizeof(m_aryH264StartCode) + frameDataLength ;
            tmpVideoFrame.m_pRawData = (unsigned char*)malloc(sizeof(m_aryH264StartCode) + frameDataLength );
            if(tmpVideoFrame.m_pRawData)
            {
                memcpy(tmpVideoFrame.m_pRawData, m_aryH264StartCode, sizeof(m_aryH264StartCode));
                memcpy(tmpVideoFrame.m_pRawData + sizeof(m_aryH264StartCode), frameData, frameDataLength);
            }
            
        }
```

再來處理 Pframe，一樣用memcpy複製。



tmpVideoFrame.m_uintVideoTimeSec = presentationTime.tv_sec;
tmpVideoFrame.m_uintVideoTimeUSec = presentationTime.tv_usec;
                
[self.m_VideoDecoder.m_FrameBuffer addFrameIntoBuffer:tmpVideoFrame];

這邊將presentationTime加入，就可以把這個frame加入frame buffer了。


if(self.m_blnIsRecording)
{
...
}


然後還有一大段處理recoding的code，但是跟我改成硬體解碼無關，這段code都沒改動到。


----------------------------------------------------------

這邊是我查到人家寫的live555專案大概流程，可以在追code時瞭解程式大綱
詳細可看：
http://blog.sina.com.cn/s/blog_616fb0880101cut0.html

1. Socket environment initial. 
2. Parse the input parameters.
3. CreateClient. Get class RTSPClient construct (return class medium and some vars)
4. Send RTSP “options“ and get OPTIONS from server.
5. Get SDP description by the URL of the server(return value:SDPstring)
6. Create media session from the SDP descriptor above.
For each subsession, RTSPClient->setupMediaSubsession(*)
7 Create output files: Only for the Receiver (Store the streaming but not play it)
For different file format, use different *FileSink class
This uses the QuickTime file as demo. Output to the ‘::stdout’ 
8. startPlayingStreams
9. env->taskScheduler().doEventLoop()
Main loop for get the data from the server and parse and store or play directly.
void FramedSource::afterGetting(FramedSource* source)
doGetNextFrame1();
-----------------------------------


-----------------------------------

一些資料網站：

OpenGL or EGL:
http://tips.hecomi.com/entry/20130223/1361628641
http://www.cnblogs.com/kiffa/archive/2013/02/21/2921123.html
http://blog.csdn.net/silangquan/article/details/8496454
http://blog.csdn.net/majiakun1/article/details/8621418
http://blog.csdn.net/d_uanrock/article/details/50518885

live555 or rtsp:
http://albert-oma.blogspot.tw/2012/05/rtp.html
http://blog.sina.com.cn/s/blog_616fb0880101cut0.html

