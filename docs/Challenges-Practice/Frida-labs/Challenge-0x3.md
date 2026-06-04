Opening the app we can see a click me button if we use frida trace we can see whats happening behind the button
 `frida-trace -U -j '`*`com.ad2001.frida0x3`*`!*' 'Frida 0x3'`
<embed src=""></embed>
it is activating onclick button in main activity if we explore it using jadx we can see that there is if else condition to load the flag
![](./images/Challenge-0x3-img-0.png)
if the conditions meet we will get the flag
if checker.code is 512 if we explore checker class 
![](./images/Challenge-0x3-img-1.png)
so we can directly set checker.code.value to 512 using our java script code 
```javascript
Java.perform(() => {
    var a = Java.use("com.ad2001.frida0x3.Checker");
    a.code.value=512;
})
```
in this case too we have to use the same command as challenge 0x2 since the text view should load before executing our function finding the process id of activity using 
`frida-ps -Ua`
`frida -U -p 4942 -l 0x3.js `