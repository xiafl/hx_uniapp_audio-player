  
## 背景  
主要应用于页面中有多个声音需要被管理的情况。  
比如： 有多个背景音乐需要切换、页面中有很多其它的声音在不同的时候进行播放(如各种按钮上的点击声音、某些提示声音等)   
如果页面中就只有简单几个声音，则没有太大的必要使用这个模块。  

其实就是的平常的游戏中声音的管理部分，可以调整音量大小，控制是否播放声音。   

## 获取
  
npm安装:  npm install @xfl/hx_uniapp_audio-player
  
## 使用方法

### 配置   
options = {  
　　PLAYBGMC: true,  //是否开启背景音乐的播放。 这个是背景音乐的总开关。  
　　PLAYEFFECT: true,  //是否开启音效的播放。 这个是音效的总开关。  
　　isCanSaveToLocal: true,  //音乐和音效的设置是否可以存储到本地。  
　　dir: 'static/',  //音频文件的路径，可以是项目中的路径，也可以是网络路径。  
　　extend: '.mp3',  //声音文件的默认扩展名 .mp3 或 .wav  
　　language: 'pt',  //普通话 或 闽南语   mn  pt, 或其它值。  
　　effectVolume: 1,  //音效(点击声音大小)   0 表示关闭(停止播放) 1表示最大声。  
　　musicVolume: 1,  //音乐大小(背景音乐声音大小)   0 表示关闭(停止播放) 1表示最大声。  
　　localAudioName: '__game_audio__', // 存储到本地的名字  
　　timeout: 5000, //超时时间 ms  
}  
    
### play方法
参数:  
play方法传入的参数:   
　　字符串被识别为 声音文件的url,   必须。  
　　数字被识别为 音量大小,          可省略，默认为1。     
　　布尔值被识别为 是否循环播放      可省略，音效默认为false, 背景音乐默认为 true。  
　　函数被识别为 是否播放成功的回调   可省略。当传入时，则play方法返回 undefined, 否则 将返回一个 promise 对象。  
因为是通过参数的类型来识别参数的，所以传入参数的顺序可以变化而不影响结果。  
播放的是否是背景音乐，是通过 url 中 是否 包含 'bgMusic' 来识别的。  
背景音乐同时只能有一个在播放，而音效则没有限制。  
当 play 方法返回 promise对象时，其中只有 then 会被回调，通过回调中的参数判断当前声音是否播放成功，catch一般是不会被回调的。
    
示例:  
play('aa.mp3'); //播放一个声音  
play('aa'); //播放一个声音，会加上 相对路径和扩展名，相当于  play('static/aa.mp3');  
play('aa'， 0.5); //播放一个普通声音, 音量为 0.5  
play('https://www.bb.com/uuu.wav'， 0.5); //播放一个普通声音, 音量为 0.5  
play('aabgMusic'， 0.5); //循环播放一个背景音乐, 音量为 0.5  
play('aa'， true); //循环播放一个声音, 音量为 1  
play('aa'， funciont(result){console.log(result)}); //播放一个声音, 传入是否成功播放的回调函数。  
play('aabgMusic'， 0.5).then((result)=>{ //使用promise的方式注册回调函数。  
　　console.log(result)  
});
  
result 的可能值:  
{errMsg: 'audio name is Empty', type: 'init'} 声音文件的地址为空  
{errMsg: 'volume is zero or not permit play', type: 'init'} 音量大小为0 或 播放的总开关设置为 false  
{errMsg: 'is playing', type: 'init'}  当前完全相同的背景音乐正在播放中    
{errMsg: 'play err', type: 'err'}  播放失败，原因有很多: 没有网络、超时、找不到资源、跨域等   
{errMsg: 'play suc', type: 'suc'}  播放成功(注意，不是播放完成)   
{errMsg: 'play finish', type: 'finish'} 当向play传入回调时，才会在 正常播放完成或播放失败或停止播放时，在callback中传入这个值。  
  
### getAudio(url)
返回相应的 innerAudioContext 对象。 可以在这个对象上注册其它的回调，或直接进行操作。  
注意，要调用过一次 play(url) 函数后，才会有这个对象，当播放完成后，会删除这个对象。  
  
### opt(type, url, volume)
操作 innerAudioContext 对象。这个函数不会提示本次操作是否成功。  
url: 注意， 这里使用 'bgMusic' 来代表背景音乐的名字，是固定写法，因为同时只能有一个背景音乐进行播放。  
type 可以是 'play' 'resume' 'pause' 'stop' 'destroy'  
volume 是改变音量大小  
   
示例:   
opt('pause', 'bgMusic', 0.3); //暂停背景音乐，并设置音量为0.3  
opt('stop', 'http://www.aaa.com/bb.mp3'); //停止这个音效的播放  
  
### clear()  
删除所有的内部存储的 innerAudioContext 对象。即所有的动态的播放信息。  
    
### saveToLocal()
将本管理器中的设置 保存到本地存储中。(仅会保存 effectVolume、musicVolume、language)  
	  
### clearLocal()
清空本地存储中的相关数据。  
	  
### getFromLocal()
从本地存储中读取数据，更新到本管理器中。  
    
## 注意事项
1. 当切换页面时，当前页面中正在播放的声音可能会被清除掉，本插件中并未考虑这个问题。  
2. play 方法中，只能得到当前的音频是否播放成功的结果，如果有需要监听声音的其它状态，则可以 使用 getAudio() 获取或手动创建音频对象进行监听。  
3. opt 方法只能操作 在 play方法中成功打开的音频对象。  
4. 只有背景音乐的播放会存有中间数据，因为一般情况音效是很短的，而背景音乐则很长。
   
## 问题反馈  
如在使用中发现bug或有优化的建议和意见，请在评论区留言。   
  