# ajax

AJAX 支持局部且异步地发送请求。

```html
<form id="Form">
<!-- ... -->
<button 
</from>
```

`<input type="button" value="发送请求" onclick="send_ajax()">`

```
script>
           function send_ajax() {
           
               $.ajax({
               	   type: "GET",
                   url:"/userInfo",
                   data:{"username":"jack","age":13},
                   success:function (date) {
                       alert(date);
                   },

                   error:function () {
                       alert("出错了!");
                   },

                   dataType:"text"
               });
           }
        </script>
```

