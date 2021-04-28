---
title: "kxmovie問題與解法分享"
date: 2021-03-03T16:17:45+08:00
tags : [ "FFMpeg", "OpenGL", "iOS"]
layout: post
type:  "post"
draft: false
---

最近一直在研究kxmovie, ffmpeg,

在此將遇到的問題與解法分享給大家。

下載與編譯ffmpeg(在mac)：
１．
根據官網的步驟來下載與編譯ffmpeg:
https://trac.ffmpeg.org/wiki/CompilationGuide/MacOSX
下面是一個日文網站，算是比較詳細的介紹官網那些步驟：
http://qiita.com/mito_log/items/4eb9049aa44631e8b0c7

如果執行以下這行時有問題：
./configure  --prefix=/usr/local --enable-gpl --enable-nonfree --enable-libass --enable-libfdk-aac --enable-libfreetype --enable-libmp3lame --enable-libopus --enable-libtheora --enable-libvorbis --enable-libvpx --enable-libx264 --enable-libxvid
可以在前面加上：
DYLD_LIBRARY_PATH=/usr/local/lib
變成：
DYLD_LIBRARY_PATH=/usr/local/lib ./configure  --prefix=/usr/local --enable-gpl --enable-nonfree --enable-libass --enable-libfdk-aac --enable-libfreetype --enable-libmp3lame --enable-libopus --enable-libtheora --enable-libvorbis --enable-libvpx --enable-libx264 --enable-libxvid

編譯完後．編譯好的檔案會在以下路徑裡：
/usr/local/lib
/usr/local/include

本方法較為麻煩，且編譯好的檔案也不是在原目錄旁，
推薦用方法2，腳本編譯。

2.本方法為腳本，下載後將ffmpeg專案丟到裡面(與build*ffmpeg.sh同一層)，然後執行腳本即可。
https://github.com/kewlbear/FFmpeg-iOS-build-script


遇到的問題：
１．問題：ffmpeg player 播放過慢，會常常卡頓。

分析原因：ffmpeg  playerg的播放流程是 1.用av_read_frame下載frame  2.avcodec_decode_video2和avcodec_decode_audio4進行decode 3.用OpenGL繪圖。這三個動作是循序執行的：1->2->3->1->2->3...
在網路不穩的情況(公司環境雜，網速漂移大)，會來不及執行１，因為此時１正在等２和３。

解決方法：
由原本的循序執行，改為非同步執行：
1,1,1,1,1,1,.......
2,2,2,2,2,2,.......
3,3,3,3,3,3,.......

新增雙重buffer機制：用來緩存１與２的結果。

微調後目前的buffer機制：
buffer1:min 30s, max 自動增大，直到收到記憶體不足警告後馬上停止增大。
buffer2:min 1s, max 5s。

2. 問題：需顯示buffer bar

分析：在目前的雙重buffer機制下，buffer bar的長度應為buffer1(frame)的長度。

解決方法：buffer bar長度代表緩衝了多長的時間，因此需將buffer1的內容轉換成時間。有兩種方法：
1.avPacket.duration: 代表此幀的時間，因此將buffer1中所有的duration加起來即為緩衝時間。
2.avPacket.pst：代表此幀的播放時間，因此buffer1中最後一幀的pst即為緩衝時間。

最後採用2，因為拖動時間軸時會清空buffer，因此用１會有問題。
2目前有些微的不準，但差異很小應可接受。

3. 問題：wmv與mkv拖動時間軸會卡住。

分析：avformat_seek_file 是用來seek影片特定位置，拖動時間軸會呼叫此函式。

解決方法：
原本code是
avformat_seek_file（_formatCtx，_videoStream，TS，TS，TS，AVSEEK_FLAG_FRAME）;
改成
avformat_seek_file（_formatCtx，_videoStream，0，TS，TS，AVSEEK_FLAG_FRAME）;

4. 問題：問題3解決後，不會卡住，但是會跳回影片一開始。
解決方法：
原本code是
if(video)
avformat_seek_file _videoStream
if(audio)
avformat_seek_file _audioStream

改為
f(video)
avformat_seek_file _videoStream
else if(audio)
avformat_seek_file _audioStream

原理未知，這樣改問題就解決了。

5.快速拖動時間軸，會導致畫面lag。
分析：每拖動一次時間軸，會seek一次，拖動太快會導致一秒內seek好幾次。
解決方法：加入timer，每１秒才觸發一次seek。

6.特殊規格的mov與mp4會非常lag，下載frame的速度很慢。

分析：http://ffmpeg.org/pipermail/libav-user/2014-October/007586.html
video stream 和 audio stream 不同步（非stream）
導致ffmpeg需要不斷發出seek請求，取得分散在檔案中資料（可能某一幀在檔案頭，下一幀在檔案尾）
這也解釋了為什麼在讀取mov時，ffmpeg的log會看到ffmpeg不斷發出請求封包。

解決方法：https://ffmpeg.org/pipermail/ffmpeg-devel/2014-March/155970.html
判斷(*s)->seekable = h->is_streamed 是否為is_streamed，特殊規格的mov與mp4為非stream.
對這些非stream的影片進行處理。

7.有時播放.mp4會沒有畫面。
解決方法：
移除以下幾行code
    formatCtx->probesize2 = 32;
    formatCtx->max_analyze_duration2 = 32;
    formatCtx->max_index_size = 10000000;
這些參數導致問產生。

8.設定time out
解決方法：
av_dict_set(&stream_opts, "timeout", "10000000", 0);
av_dict_set(&stream_opts, "stimeout", "10000000", 0);

9.播放到一半，按back導致crash
分析：按back會進行釋放，因此播到一半被釋放會導致讀取錯誤。
解決方法：
需嚴謹的將播放和釋放進行同步處理，確保釋放後不會再被讀取到。
1.pthread_mutex_lock
2.@synchronized


待解決的問題：
１．問題：DV(digital video)播放速度過快。
推測可能與time_base換算有關。

２．問題：拖動時間軸時清空buffer後需重新下載，buffer作用小。
可能解法：從雙重buffer改為三重buffer，多加一層sd card/disk buffer。
將buffer訊息存入硬碟而不是清空，如此就不需要重新下載，下載完後切斷網路也可播放自如。

目前推測需要用到兩種技巧：
1.ffmpeg只接收url(網址和檔案路徑)，可能需先改成直接從buffer中讀取：
http://my.oschina.net/michaelyuanyuan/blog/66275
http://bbs.chinavideo.org/forum.php?mod=viewthread&tid=15548

2.將frame存入disk與從disk取出frame：
https://gist.github.com/sunjun/b386c314757651baafb7
