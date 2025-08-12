# 鼠标点击事件

方式一：使用onclock

```js
<scrip>
<button id = "captcha-refresh" type = "button" class ="refresh-btn" onclick="refreshCaptcha()" >刷新</button>
 function refreshCaptcha(){
            const captchaImg = document.getElementById("captcha-image");
            {# 增加时间戳，防止缓存#}
            captchaImg.src =CAPTCHA_URL + '?t='+ new Date().getTime();
            {# 清空输入框和信息 #}
            document.getElementById("captcha-image").value='';
        }
</scrip> 
```

方式二：添加监听事件

```js
<scrip>
<button id = "captcha-refresh" type = "button" class ="refresh-btn"  >刷新</button>
 function refreshCaptcha(){
            const captchaImg = document.getElementById("captcha-image");
            {# 增加时间戳，防止缓存#}
            captchaImg.src =CAPTCHA_URL + '?t='+ new Date().getTime();
            {# 清空输入框和信息 #}
            document.getElementById("captcha-image").value='';
        }
</scrip>  
{# 添加监听事件#}
        document.getElementById("captcha-refresh").addEventListener('click',refreshCaptcha)
```

