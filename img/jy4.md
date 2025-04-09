# 极验4 验证码 （2025/2/24）

抓包分析

进入网页后，打开开发者人员工具进行[抓包](https://so.csdn.net/so/search?q=抓包&spm=1001.2101.3001.7020)，此时还未点击按钮开始验证，抓到了一个名为 `load?callback=xxx` 的包，`载荷` 包含了一些参数：

![image-20250224142635192](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/20250409194736391.png)

- callback ：geetest_ + 时间戳
- captcha_id ： 验证码 id 固定值，后文分析
- challenge ：动态变化，后文分析
- client_type ：客户端类型，这里为web端
- risk_type ：验证码类型，滑块：slide
- lang ：语言



响应的内容

![image-20250224143442274](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224143442274.png)

- lot_number ：w参数加密中需要用到的参数，后面验证中需要
- slice ：滑块图片地址
- bg ：缺口背景图片地址
- pow_detail ：w参数加密中用到
- payload ：验证中参数
- process_token ：验证中参数



点击按钮，拖动滑块验证成功，`/verify?callback= xxx` 验证网址

![image-20250224144450867](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224144450867.png)

- w ：js 加密获得

响应内容，result 为 success 为验证通过，fail 为验证失败

![image-20250224144636407](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224144636407.png)



### 逆向分析

**captcha_id**

直接先搜关键字，会发现 captcha_id 为固定值 54088bb07d2df3c46b79f80300b0abbe

![image-20250220145906437](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250220145906437.png)

**challenge**

直接搜索关键字

![image-20250220151814552](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250220151814552.png)



然后我们发现在uuid()中生成了，我们跟进将其扣下即可

![image-20250220152012392](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250220152012392.png)

**w 参数**

先全局搜索，我们会在 gcaptcha4.js 中发现，我们可以在这打上断点，验证即可

![image-20250224145816352](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224145816352.png)

往上看，我们可以看到 w 参数加密的地方，参数如图，第二个参数为一个对象，可以追进去看

![image-20250224150324539](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224150324539.png)



![image-20250224150624928](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224150624928.png)

追进加密的函数会发现第二个参数只用到了 options json 中的 pt 参数，其他参数都没有用到，这里可以直接写死

```js
_ᖁᕿᕶᕵ = {
    'options':{
    'pt': '1'
    }
}
```

![image-20250224151139975](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224151139975.png)

第一个参数先是经过了 stringify 函数，这里我们就可以直接用 JSON.stringify

![image-20250224151751020](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224151751020.png)

我们跟进加密方法，直接用 window 将方法导出来

![image-20250224152228734](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224152228734.png)

好了，加密的方法导出来了，我们将 gcaptcha4.js 整个文件代码复制下来，进行补环境。

这里我用的是 v 佬的自动吞吐环境工具 v_tools 插件

![image-20250224152648012](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224152648012.png)

![image-20250224152745585](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224152745585.png)

将生成的代码粘贴下来，即可运行

![image-20250224154536830](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224154536830.png)

那么，接下来分析第一个参数

![image-20250224154825053](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224154825053.png)

- setLeft ：滑块滑动的距离
- passtime ：滑动的时间
- lot_number ：第一个 `/load`请求中获取
- pow_msg ：load请求返回中的pow_detail组成  version|bits|hashfunc|datetime|captcha_id|lot_number||？
- geetest ：可以写死
- lang ：语言，可以写死
- ep ：可以写死
- biht ：可以写死
- gee_guard ：可以写死
- HufC ：可以写死

我们在加密的位置往上看

em：

![image-20250221150724110](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250221150724110.png)

![image-20250221150851252](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250221150851252.png)

可以从这里得到，这里也可以写死



开头的json是动态的，每次都不一样，可以是看到在这里生成的，我们追进去看一下

![image-20250221151830721](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250221151830721.png)

![image-20250221163649289](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250221163649289.png)

![image-20250221163831326](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250221163831326.png)

发现是一个通过索引截取字符串的函数，截取的是lot_number ,我去扣算法老有地方报错，这里我们自己截取吧

```js
function getStringByIndexes(lotNumber) {
        return lotNumber.substring(22, 24) + lotNumber.substring(19, 21) + '.' + lotNumber.substring(6, 12)
    }

    var i = getStringByIndexes(lot_number)
    var r = lot_number.substring(23, 31)
    var a = i.split('.')
    var b = `"${a[0]}":{"${a[1]}":"${r}"}`
```

userresponse ：

我们向上追栈，会发现在 $_BGIK 处产生了 userresponse 
_ᖈᖈᖆᕿ：滑动距离  _ᕸᕷᖘᖗ[_ᖀᕷᖙᕶ(1445)] 为定值：1.0059466666666665

![image-20250224160811627](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224160811627.png)



pow_msg，pow_sign：

我们接着向上追栈，追到init这里，会发现pow_msg，pow_sign的产生，我们在上面打上断点

![image-20250221174807307](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250221174807307.png)

在这里发现产生了

![image-20250221175054727](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250221175054727.png)

我们跟进去，就能看到pow_msg最后那串字符串的产生，跟进去

![image-20250221175617365](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250221175617365.png)

会发现跟极验3中的那个随机字符串一样，直接扣下来

![image-20250221175750383](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250221175750383.png)

然后接着往后走，我们发现 pow_sign 是将 pow_msg 通过 MD5 加密的，我们将加密的参数在在线网站上加密一下，发现结果一样，所以这里的 MD5 加密没有魔改，我们 JS 中直接用库就行

![image-20250224093902065](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224093902065.png)

![image-20250224094115457](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224094115457.png)

然后就是将字符串进行拼接，加密

验证成功

![image-20250224135859644](https://raw.githubusercontent.com/yunxirumo/PicGo/main/img/image-20250224135859644.png)