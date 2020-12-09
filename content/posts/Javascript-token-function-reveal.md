---
title: Javascript 逆向找到 genToken() 源码
date: 2020-04-08 19:36:32
tags:
 - Javascript
categories:
 - 技术
---

最近开始了我的实习之旅，爬虫相关的， 然后就遇到某个网站在获取部分url的时候, header里会带有一个token, 根据内容能发现是hmac加密的

没有token的时候一律会返回错误， 于是研究了下JS的逆行，找到了生成token的function

理念就是打断点，先是针对于url的断点，找到是在哪个js文件中发生了http请求

![](https://i.loli.net/2020/04/08/SOp8XdqNaih7CLD.png)

然后顺着call stack一层一层往上找，看在哪里生成的header

![](https://i.loli.net/2020/04/08/cflYktUxpgHB9QG.png)

![](https://i.loli.net/2020/04/08/CpcIrAsE3jaxdNl.png)

然后在getAuthToken()这里打点, 能在环境变量里看到用于加密使用的key

![](https://i.loli.net/2020/04/08/6oOEAYNVPmzwXRr.png)

然后顺着找到getAuthToken()的源码, 

```javascript
function (t){
    var e=s.initConfig.getConfig("env"),
        n=this.authKeys[e],
        r=s.initConfig.getTimeOffset(),
        i=(new Date).getTime()+r,
        a=(t&&t.authAcl||"")+"/*",
        u=(0,o.default)({},this.akamaiAuthconfig,{key:n,time:i,acl:a});
    return new c.default(u).generateToken()
}
```

以及`c.default(u).generateToken()`的源码

```javascript
    var t = this.getIpField() + this.getStartTimeField() + this.getExprField() + 
            this.getAclField() + this.getSessionIdField() + this.getDataField(),
    e = t + "" + this.getUrlField() + this.getSaltField(),
    n = RegExp(this.delimiter + "$");
    return t + "hmac=" + u.default.HmacSHA256(e.replace(n, ""), function(t) {
        var e = "",
        n = 0;
        do {
            var r = t[n] + t[n + 1],
            o = parseInt((r + "").replace(/[^a-f0-9]/gi, ""), 16);
            e += String.fromCharCode(o), n += 2
        } while (t.length > n);
            return e
        }(this.key))
```

再根据实际需要进行简化, 并进行比对，确认这个function正确运行.

```Python
    def gen_token(self):
        def f(t):
            e, n = "", 0
            while(len(t) > n):
                r = t[n] + t[n+1]
                e += chr(int(re.sub(r"[^a-f0-9]", "", r+"") or "0", 16))
                n += 2
            return e
        key = ""
        t = int(time.time())
        i = 6000
        e = f"st={t}~exp={t+i}~acl=/*"
        _hash = hmac.new(f(key).encode('latin-1'),
                         msg=e.encode('latin-1'), digestmod=sha256).hexdigest()
        return f"{e}~hmac={_hash}"
```

