> * WebRTC，名称源自网页即时通信（英语：Web Real-Time Communication）的缩写，是一个支持网页浏览器进行实时语音对话或视频对话的API

> * ios 需要使用type='file' 的方式上传录屏文件，此api在ios设备存在兼容
***

## [getUserMedia](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaDevices/getUserMedia) 会提示用户给予使用媒体输入的许可，媒体输入会产生一个MediaStream


1. 检查设备是否支持
```javascript
   hasGetUserMedia(){
    try {
        return !!(navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia || navigator.msGetUserMedia || navigator.oGetUserMedia || (navigator.mediaDevices && navigator.mediaDevices.getUserMedia));
    } catch (u) {
        return false;
    }
   }
```

2. 设备兼容方法
```javascript
   setUserMedia(){
    // 老的浏览器可能根本没有实现 mediaDevices，所以我们可以先设置一个空的对象
    if (navigator.mediaDevices === undefined) {
        navigator.mediaDevices = {};
    }

    // 一些浏览器部分支持 mediaDevices。我们不能直接给对象设置 getUserMedia
    // 因为这样可能会覆盖已有的属性。这里我们只会在没有getUserMedia属性的时候添加它。
    if (navigator.mediaDevices.getUserMedia === undefined) {
        navigator.mediaDevices.getUserMedia = function(constraints) {

            // 首先，如果有getUserMedia的话，就获得它
            var getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia || navigator.msGetUserMedia || navigator.oGetUserMedia;

            // 一些浏览器根本没实现它 - 那么就返回一个error到promise的reject来保持一个统一的接口
            if (!getUserMedia) {
                return Promise.reject(new Error('getUserMedia is not implemented in this browser'));
            }

            // 否则，为老的navigator.getUserMedia方法包裹一个Promise
            return new Promise(function(resolve, reject) {
                getUserMedia.call(navigator, constraints, resolve, reject);
            });
        }
    }
   }
```

3. 获取设备媒体流
```javascript
    getUserMedia(){
        navigator.mediaDevices.getUserMedia({ audio: true, video: true })
        .then(function(stream) {
        /* 使用这个stream stream */
            // 旧的浏览器可能没有srcObject
            if ("srcObject" in video) {
                video.srcObject = stream;
            } else {
                // 防止在新的浏览器里使用它，应为它已经不再支持了
                video.src = window.URL.createObjectURL(stream);
            }
            video.onloadedmetadata = function(e) {
                video.play();
            };
        })
        .catch(function(err) {
        /* 处理error */
         console.log(err.name + ": " + err.message);
        });
    }
```

> * 部分webview内嵌场景，需要设置video音量属性为静音播放才能显示摄像头  
> set `video.muted=true` and `video.volume=0` 

> * 部分手机如荣耀Magic设备，constraints需要设置 `video: true`,才能正常显示摄像头

> * 另外 oppo k系列(k3,k7)存在摄像头黑屏，在play前能闪现正常的问题，还在研究中

***

## [recordRTC](https://recordrtc.org/) WebRTC JavaScript Library for Audio+Video+Screen+Canvas (2D+3D animation) Recording

1. 获取recorder
```javascript
    getMediaRecorder(callback){
        try {
            mediaRecorder = RecordRTC(window.stream, {
                'type': 'video'
            });
            webRTCStartRecording(callback);
        } catch (err) {
            /* 处理error */
         console.log(err.name + ": " + err.message);
        }
    }
```

2.媒体流记录过程
```javascript
    webRTCStartRecording(callback){
        try {
            mediaRecorder.startRecording();

            const sleep = m => new Promise(r => setTimeout(r, m));
            await sleep(3000);

            mediaRecorder.stopRecording(function(blobURL) {
                //获取成功回调函数，blob即为音频文件
                //   // audioURL为生成音频的URL地址，通过window.open(audioURL)可以实时预览
                //   window.open(audioURL);
                //   var recordedBlob = mediaRecorder.getBlob();
                //   // 转换为base64
                //   mediaRecorder.getDataURL(function(dataURL) {
                //   });
                readBlob()
                stopStream();
                callback();
            });
        } catch (e) {
            stopStream();
            callback();
        }
    }
```
3.red
```javascript
    readBlob(){
        var recordedBlob = mediaRecorder.getBlob();
        var reader = new FileReader();
        reader.readAsDataURL(recordedBlob);
        reader.onload = function(event) {
            callback(event.target.result);
        };
    }
```
4. 多个视频轨道逐一关闭
```javascript
    stopStream(){
            window.stream.getVideoTracks().concat(window.stream.getAudioTracks()).forEach(function(track) {
                track.stop();
            });
    }
```

> * [some demo](https://www.webrtc-experiment.com/RecordRTC/simple-demos/)