Initially when we try to install the apk it is not signed we use apktool to sign the apk
After installing the apk in to genymotion we can find out the app is waiting for a password and if we load the apk into jadx we cant find much so we unzip the apk using apktool `apktool -d cryptovault_signed.apk `and we go to assests folder and we can find index.html and many other java script files since there is only 1 html file it should be the main and we explore it using vs code and find out the password to valut in the function unlockvalut()
![](./images/Cryptovault-img-0.png)
then we get a hexadecimal string of digits
![](./images/Cryptovault-img-1.png)
we copy it and use cyberchef but it doesnt give a proper result so then we go to code again and we can find a function retrieve encrypted key and it called keygen() finction which is located in keygen.js so if we open it the code is obfuscated like below
```javascript
(function(_0x506dbf,_0x170411){const _0x12e004=a0_0x1707,_0x3fbe25=_0x506dbf();while(!![]){try{const _0x3b5636=parseInt(_0x12e004(0x122))/0x1*(parseInt(_0x12e004(0x117))/0x2)+-parseInt(_0x12e004(0x111))/0x3*(-parseInt(_0x12e004(0x121))/0x4)+-parseInt(_0x12e004(0x11b))/0x5*(parseInt(_0x12e004(0x11f))/0x6)+parseInt(_0x12e004(0x113))/0x7*(-parseInt(_0x12e004(0x11d))/0x8)+parseInt(_0x12e004(0x125))/0x9*(parseInt(_0x12e004(0x11e))/0xa)+-parseInt(_0x12e004(0x123))/0xb+parseInt(_0x12e004(0x120))/0xc*(parseInt(_0x12e004(0x112))/0xd);if(_0x3b5636===_0x170411)break;else _0x3fbe25['push'](_0x3fbe25['shift']());}catch(_0x18c02d){_0x3fbe25['push'](_0x3fbe25['shift']());}}}(a0_0x32dd,0x4ff3a));function a0_0x32dd(){const _0xb65be8=['9425749445e494332757363353f5d6f50353b79445d7336343270373270366f586365753f546c60336f5','length','map','38495LKnOYO','substr','8lZAZpw','6486450oYKfNK','402RerQLO','12MNesgS','4FulGyI','528939ZPevpd','861608xHrljL','split','9gQnkOh','toString','242571ENkSLa','502515FcoXSF','2628171KytvIJ','push','slice','join','2HiwuOL'];a0_0x32dd=function(){return _0xb65be8;};return a0_0x32dd();}function affineEncrypt(_0x1930bc,_0x36e79b,_0x33477e){return(_0x36e79b*_0x1930bc+_0x33477e)%0x100;}function xor(_0x3a38fa,_0x3c3309){return _0x3a38fa^_0x3c3309;}function a0_0x1707(_0x3d4d4c,_0x35b685){const _0x32dd9d=a0_0x32dd();return a0_0x1707=function(_0x170770,_0x4c15fe){_0x170770=_0x170770-0x110;let _0x3e6dad=_0x32dd9d[_0x170770];return _0x3e6dad;},a0_0x1707(_0x3d4d4c,_0x35b685);}function hexToBytes(_0x1d9eb0){const _0x3e7222=a0_0x1707;let _0x2ac99a=[];for(let _0x2363dc=0x0;_0x2363dc<_0x1d9eb0[_0x3e7222(0x119)];_0x2363dc+=0x2){_0x2ac99a[_0x3e7222(0x114)](parseInt(_0x1d9eb0[_0x3e7222(0x11c)](_0x2363dc,0x2),0x10));}return _0x2ac99a;}function reverseString(_0x22dcba){const _0x102ddd=a0_0x1707;return _0x22dcba[_0x102ddd(0x124)]('')['reverse']()[_0x102ddd(0x116)]('');}function keygen(){const _0x588caa=a0_0x1707;let _0x620410=_0x588caa(0x118),_0x19eb60=[_0x620410[_0x588caa(0x115)](0x0,0xe),_0x620410[_0x588caa(0x115)](0xe,0x1c),_0x620410[_0x588caa(0x115)](0x1c,0x2a),_0x620410[_0x588caa(0x115)](0x2a,0x38),_0x620410['slice'](0x38,0x46),_0x620410[_0x588caa(0x115)](0x46,0x54)],_0x4c2f5e=[_0x19eb60[0x3],_0x19eb60[0x5],_0x19eb60[0x1],_0x19eb60[0x4],_0x19eb60[0x2],_0x19eb60[0x0]],_0x22e526=reverseString(_0x4c2f5e['join']('')),_0x2051e9=hexToBytes(_0x22e526),_0x3788f1=0x9,_0x2afabe=0x7,_0x56285d=0x33,_0x351569=_0x2051e9['map'](_0x585a6f=>xor(affineEncrypt(_0x585a6f,_0x3788f1,_0x2afabe),_0x56285d));return _0x351569[_0x588caa(0x11a)](_0x5ca89b=>('0'+_0x5ca89b[_0x588caa(0x110)](0x10))[_0x588caa(0x115)](-0x2))[_0x588caa(0x116)]('');}
```
so we use deobfuscator and the code looks like below 
```javascript
function affineEncrypt(_0x1930bc, _0x36e79b, _0x33477e) {
  return (_0x36e79b * _0x1930bc + _0x33477e) % 0x100;
}
function xor(_0x3a38fa, _0x3c3309) {
  return _0x3a38fa ^ _0x3c3309;
}
function hexToBytes(_0x1d9eb0) {
  let _0x2ac99a = [];
  for (let _0x2363dc = 0x0; _0x2363dc < _0x1d9eb0.length; _0x2363dc += 0x2) {
    _0x2ac99a.push(parseInt(_0x1d9eb0.substr(_0x2363dc, 0x2), 0x10));
  }
  return _0x2ac99a;
}
function reverseString(_0x22dcba) {
  return _0x22dcba.split('').reverse().join('');
}
function keygen() {
  let _0x19eb60 = ["9425749445e494332757363353f5d6f50353b79445d7336343270373270366f586365753f546c60336f5".slice(0x0, 0xe), "9425749445e494332757363353f5d6f50353b79445d7336343270373270366f586365753f546c60336f5".slice(0xe, 0x1c), "9425749445e494332757363353f5d6f50353b79445d7336343270373270366f586365753f546c60336f5".slice(0x1c, 0x2a), "9425749445e494332757363353f5d6f50353b79445d7336343270373270366f586365753f546c60336f5".slice(0x2a, 0x38), "9425749445e494332757363353f5d6f50353b79445d7336343270373270366f586365753f546c60336f5".slice(0x38, 0x46), "9425749445e494332757363353f5d6f50353b79445d7336343270373270366f586365753f546c60336f5".slice(0x46, 0x54)];
  let _0x4c2f5e = [_0x19eb60[0x3], _0x19eb60[0x5], _0x19eb60[0x1], _0x19eb60[0x4], _0x19eb60[0x2], _0x19eb60[0x0]];
  let _0x22e526 = _0x4c2f5e.join('').split('').reverse().join('');
  let _0x2051e9 = hexToBytes(_0x22e526);
  let _0x351569 = _0x2051e9.map(_0x585a6f => (0x9 * _0x585a6f + 0x7) % 0x100 ^ 0x33);
  return _0x351569.map(_0x5ca89b => ('0' + _0x5ca89b.toString(0x10)).slice(-0x2)).join('');
}
```
so we found out this is the function which generates the hexadecimal string so we reverse the exact same process and we are going to use python 
```python
import string

def mod_inv(a, m):
    t, new_t, r, new_r = 0, 1, m, a
    while new_r:
        q = r // new_r
        t, new_t = new_t, t - q * new_t
        r, new_r = new_r, r - q * new_r
    if r > 1: raise ValueError(f"{a} has no inverse mod {m}")
    return t + m if t < 0 else t

hex_str = '9425749445e494332757363353f5d6f50353b79445d7336343270373270366f586365753f546c60336f5'
key = [(mod_inv(9, 0x100) * (byte - 7)) % 0x100 ^ 0x33 for byte in [int(''.join([hex_str[i:i+14] for i in range(0, len(hex_str), 14)][i] for i in [3, 5, 1, 4, 2, 0])[::-1][i:i+2], 16) for i in range(0, len(''.join([hex_str[i:i+14] for i in range(0, len(hex_str), 14)][i] for i in [3, 5, 1, 4, 2, 0])[::-1]), 2)]]

result = ""
for i in range(42):
    for c in string.printable:
        if (mod_inv(9, 0x100) * (ord(c) - 7) % 0x100 ^ 0x33) == key[i]:
            result += c
            break
print(f"Flag: {result}")
```
and the result is INTIGRITI\{50_much_f0r_53cur3_c0ld_570r463\}
