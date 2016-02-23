# android-live-sdk

在调用系统相机功能时，需要绑定SurfaceView，绑定时，该SurfaceView如果没有被系统渲染完全，则会导致绑定失败，无法正常启用相机。比如，如果在布局文件中将视频区域的布局可见性设置为“gone”，则直到第一次视频直播前这部分布局都不会被渲染，这样，第一次直播会失败，而第二次再点击视频直播时由于该部分布局已经设置“visible”过，所以不会再出现这个问题。所以再调用相机绑定前，应确保视频直播的区域已经被渲染过，将布局设置可见性为“visible”或者“invisible”。  

当点击悬浮条进入直播间时，即使视频直播区域的可见性不是“gone”也会有直播失败的情况，原因是如果当前正处于直播中，一进入直播间就会调用直播的函数，而此时，直播间里播放视频的自定义的view并没有渲染完全，也会导致绑定失败。解决方法是调用对应view的post或者postDelay方法:  
```
chatTopView.post(new Runnable() {  
    @Override  
    public void run() {  
    //查询直播状态       
    }  
});
```   
run方法执行的时候view已经渲染完全，调用视频直播就不会有问题了。但是view的post方法并不总是100%执行，为了确保该部分方法至少被执行一次，可以再添加一个500ms的timerTask来检查该方法是否被执行。  

怎么检测view是否已被渲染完？  
可以通过获取view的高度和宽度来检测view是否被渲染，当view没有被渲染，其宽度和高度是0，绑定view是会失败的。但是即使获取的高度和宽度并不是0，也不能确保绑定一定成功，因为还是有没有渲染完全的可能。自定义view的onFinishInflate方法和getViewTreeObserver().addOnGlobalLayoutListener()方法也不能确保view被完全渲染。目前找到的比较可靠的方法就是view的post方法，当view开始执行该方法时，view已经获取了它需要的所有资源。
