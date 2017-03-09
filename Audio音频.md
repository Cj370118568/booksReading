#Audio音频播放
-

###System Sounds系统声音
以播放系统声音的方式播放音频是音频播放最简单的方式。系统声音是Audio Toolbox framework下的System Sound Services定义的。播放系统声音要先```import AudioToolbox```

iOS9对系统声音的API进行了一次修改。播放系统声音要调用两个非常相似的C函数的其中一个：
	
+ AudioServicesPlayAlertSound

	如果用户没有在设置中关闭震动，在iPhone上会震动。
+ AudoServicesPlaySystemSound 

	除非你选择了```kSystemSoundID_Vibrate```作为参数，否则不会震动。

要播放的声音文件必须是没压缩的AIFF或者WAV（或者是苹果的CAF格式）

播放系统声音之前你需要一个SystemSoundID，这种id通过调用AudioServicesCreateSystemSoundID这个方法获得，下面是示例代码：

```Swift 
//iOS9之前
let sndurl = Bundle.main.url(forResource:"test", withExtension: "aif")!
        var snd : SystemSoundID = 0
        AudioServicesCreateSystemSoundID(sndurl as CFURL, &snd)
        
        AudioServicesAddSystemSoundCompletion(snd, nil, nil, {
            sound, context in
            
            AudioServicesRemoveSystemSoundCompletion(sound)
            AudioServicesDisposeSystemSoundID(sound)
            }, nil)
        AudioServicesPlaySystemSound(snd)
```

```Swift
//iOS9之后
let sndurl = Bundle.main.url(forResource:"test", withExtension: "aif")!
        var snd : SystemSoundID = 0
        AudioServicesCreateSystemSoundID(sndurl as CFURL, &snd)
        AudioServicesPlaySystemSoundWithCompletion(snd) {
            AudioServicesDisposeSystemSoundID(snd)
            print("finished!")
        }
```
	
###Audio Session
设备上播放的所有音频都由media services daemon来控制，因此你的app音频播放可能被外界的因素或者其他app打断。要播放音频的话得先决定一个策略并告诉media services daemon，例如：

+ 锁屏得时候声音是不是要停止播放？
+ 播放得声音是要被其他声音打断还是说跟别的声音一起播放？

你要通过AVAudioSession的单例audio session来告诉audio services daemon这些信息。AVAudioSession会在app启动时创建，AVAudioSession时AV Foundation framework里面的内容，你必须```import AVFoundation```你可以通过AVAudioSession.sharedInstance来得到这个单例。

通过setCategory(_:mode:options:)这个方法来定义你的播放策略。

系统自带的基础音频播放策略有：

+ Ambient（AVAudioSessionCategoryAmbient）

	你的app会跟其他app一起播放音频，会受到静音设置影响，锁屏时会停止播放。

+ Solo Ambient(AVAudioSessionCategorySoloAmbient,默认值)

	你的app会使别的app暂停播放，会受到静音设置影响，锁屏时停止播放。
	
+ Playback(AVAudioSessionCategoryPlayback)

	你的app会使别的app暂停播放，不会受到静音设置影响，如果你没有设置后台播放模式，锁屏时停止播放。
	
iOS9开始，你可以通过audio session的availableCategories属性查看设备是否支持你当前所选的策略。

options参数允许你修改播放策略（AVAudioSessionCategoryOptions）:
	
+ Mixable audio(.mixWithOthers)
+ Ducking audio(.duckOthers)
+ Mixable except for speech(.interruptSpokenAudioAndMixWithOthers)

实际开发过程中，我们经常在application(_:didFinishLaunchingWithOptions）这个方法中初始化声音播放策略，如果需要，可以在后面再进行修改。

```Swift
func application(_ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions:
    [UIApplicationLaunchOptionsKey : Any]?) -> Bool {
        try? AVAudioSession.sharedInstance().setCategory(
            AVAudioSessionCategoryAmbient)
        return true
} 
```
###Activation

###Ducking

###Interruptions

###Secondary Audio

###Routing Changes

###Audio Player

###Remote Control of Your Sound
你的app可以处理

###Playing Sound in the Background

###AVAudioEngine

###MIDI Playback

###Text to Speech

###Speech to Text

###Futher Topics in Sound 其他问题
+ Other audio session policies
+ Recording sound
+ Audio queues
+ Extended Audio File Services
+ Audio Converter Services
+ Streaming audio
+ Audio units
