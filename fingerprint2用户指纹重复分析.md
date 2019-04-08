**fingerprint2** 一款开源设备指纹采集器,在github上有**7k**的Star，看起来是那么的让人放心，今天聊一聊我们在使用这个库中猜到的坑。

本篇所讲的fingerprint2版本为**2.0.6**

**生成的指纹大面积重复问题！！！**

**生成的指纹大面积重复问题！！！**

**生成的指纹大面积重复问题！！！**

重要的问题讲三次。


## fingerprint2会取的设备信息
<font color='red'>*</font>获取不到值时返回： not available 
<font color='red'>#</font>获取不到值时返回： error
1. `userAgent`:navigator.userAgent
1. `language` : 语言
1. `colorDepth`: 返回目标设备或缓冲器上的调色板的比特深度 [screen.colorDepth](https://developer.mozilla.org/en1.US/docs/Web/API/Screen/colorDepth) <font color='red'>*</font>
1. `deviceMemory`: 以千兆字节为单位返回设备内存量。该值是通过舍入到最接近的2的幂并将该数除以1024而给出的近似值。[链接](https://developer.mozilla.org/en1.US/docs/Web/API/Navigator/deviceMemory)  <font color='red'>*</font>
1. `pixelRatio`: 像素比 [devicePixelRatio](https://developer.mozilla.org/en1.US/docs/Web/API/Window/devicePixelRatio)  <font color='red'>*</font> 
1. `hardwareConcurrency`:[navigator.hardwareConcurrency](https://developer.mozilla.org/en1.US/docs/Web/API/NavigatorConcurrentHardware/hardwareConcurrency)返回可用于运行在用户的计算机上的线程的逻辑处理器的数量 <font color='red'>*</font> 
1. `screenResolution`: 检测屏幕宽高，并根据屏幕方向矫正返回值`[width,height]`
1. `availableScreenResolution`:返回屏幕分辨率`[width,height]`，无头浏览器无法获取。<font color='red'>*</font> 
1. `timezoneOffset`: 返回从当前区域设置（主机系统设置）到UTC的时区差异（以分钟为单位）[链接](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getTimezoneOffset)
1. `timezone`:时区 <font color='red'>*</font> 
1. `sessionStorage`: 是否支持sessionStorage，不支持时返回错误 <font color='red'>#</font> 
1. `localStorage`: 是否支持localStorage <font color='red'>#</font> 
1. `indexedDb`:是否支持indexedDb <font color='red'>#</font> 
1. `addBehavior`:此时可能未定义body或以编程方式删除
1. `openDatabase`: 返回是否支持[Web SQL](http://www.runoob.com/html/html5-web-sql.html)
1. `cpuClass`:返回浏览器系统的 CPU 等级,一般无法获取 <font color='red'>*</font> 
1. `platform`: 返回表示浏览器平台的字符串,该规范允许浏览器始终返回空字符串，因此不要依赖此属性来获得可靠的答案.[链接](https://developer.mozilla.org/en-US/docs/Web/API/NavigatorID/platform) <font color='red'>*</font> 
1. `doNotTrack`: 返回用户的“不跟踪”设置。如果用户请求不被网站，内容或广告跟踪，则为“1”。一般结果为<font color='red'>*</font> 。
1. `plugins`:返回浏览器安装的插件列表。<font color='red'>*</font> 
1. `canvas`: 如果浏览器支持canvas则返回生成baes64数据。<font color='red'>*</font> 
1. `webgl`:返回浏览器对webgl绘图协议的支持情况汇总 <font color='red'>*</font> 
1. `webglVendorAndRenderer`: 返会显卡型号相关信息 <font color='red'>*</font> 
1. `adBlock`:返回是否安装去广告插件。
1. `hasLiedLanguages`: 返回用户是否改变了首选语言
1. `hasLiedResolution`:返回用户是否改变了分辨率
1. `hasLiedOs`:返回用户是否改变了操作系统
1. `hasLiedBrowser`:返回用户是否改变了浏览器
1. `touchSupport`: 返回最大触摸点数，是否支持touch，是否支持ontouchstart事件]
 1. `fonts`:返回从64种字体种筛选出的可用字体
1. `fontsFlash`:Flash字体枚举,如果没有swfobject，不会触发。
1. `audio`: 返回音频指纹
1. `enumerateDevices`:[navigator.mediaDevices](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/enumerateDevices) 请求可用媒体输入和输出设备的列表，例如麦克风，相机，耳机等


## 工作逻辑
1. 取到以上值后[数组]，将数组转为值字符串
2. 将取到的字符串做为key 传入`x64hash128`方法，生成指纹

## 指纹重复原因
`x64hash128`算法是固定的，所以在key相同的时,生成的指纹是相同的。
fingerprint2在**手机上重复的概率会更高**,绝大多数用户不会去修改手机的配置，所以重复指纹主要在发生在**同一型号**的产品。



## 推荐解决生成指纹重复方案
因为我们主要面对移动终端用户，所以fingerprint2生成的值出现大面积重复(实践中的血与泪)。
- 通过接口获取用户ip值
- FingerPrint2继续使用默认配置,在传入的`key`中手动添加`Ip`条件;
  ```js
  Fingerprint2.get(components=>{
       components.push({
          key:'ip',
          value:'192.168.1.1' //通过接口获取的到ip
        });
      let murmur = Fingerprint2.x64hash128(components.join(""), 31); //生成指纹信息
  })
  ```
加入ip信息，可以在很大程度上规避同型号的产品生成指纹信息相同的场景，切记不是百分之百。比如同一型号设备在同一wifi下生成的指纹信息也是有很大概率相同的。
## 结言
现在浏览器提供的设备信息越来越少，跟踪用户信息这只是一个思路，如果大家有奇技淫巧，欢迎交流。
