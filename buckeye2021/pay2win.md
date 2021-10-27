# pay2win

## Question

Kyle started an online magazine (The Daily Kyle) and published one of my articles on his site. Don't worry, the article literally contains the flag in plaintext, but if you want to read it you'll have to figure out how to bypass the paywall.

URL: https://pay2win.chall.pwnoh.io

## Solution
This website has a number of protections against interacting with the website or using devtools. 

Fortunately the source code is not obfuscated and we can see it uses a [JavaScript function](https://pay2win.chall.pwnoh.io/main.js) to dynamically generate a bunch of span elements for each character of the flag:

```javascript
function plantFlag () {
  const ciphertext = [234, 240, 234, 252, 214, 236, 140, 247, 173, 191, 158, 132, 56, 4, 32, 73, 235, 193, 233, 152, 125, 19, 19, 237, 186, 131, 98, 52, 186, 143, 127, 43, 226, 233, 126, 15, 225, 171, 85, 55, 173, 123, 21, 147, 97, 21, 237, 11, 254, 129, 2, 131, 101, 63, 149, 61]
  const plaintext = ciphertext.map((x, i) => ((i * i) % 256) ^ x ^ 0x99)

  const flagElement = document.querySelector('#flag')
  plaintext.map((x, i) => {
    const span = document.createElement('span')
    span.classList.add(`flag-char-${i}`)
    span.textContent = String.fromCharCode(x)
    flagElement.appendChild(span)
    return span
  })

  const flagOverlay = document.querySelector('#flag-overlay')
  flagOverlay.addEventListener('mouseover', async () => {
    await swal(flagAlert)
  })
}
```

And then uses [CSS](https://pay2win.chall.pwnoh.io/main.css) to order each of these characters:
```css
.flag-char-0 {
  order: 13;
}

.flag-char-1 {
  order: 47;
}

.flag-char-2 {
  order: 40;
}

.flag-char-3 {
  order: 49;
}

.flag-char-4 {
  order: 39;
}
...
```

I constructed the following Python function to replicate this rendering logic, which outputs the flag `buckeye{h0ly_sh1t_wh4t_th3_h3ck_1s_th1s_w31rd_ch4ll3ng3}`:
```python
import re, requests

ciphertext = [234, 240, 234, 252, 214, 236, 140, 247, 173, 191, 158, 132, 56, 4, 32, 73, 235, 193, 233, 152, 125, 19, 19, 237, 186, 131, 98, 52, 186, 143, 127, 43, 226, 233, 126, 15, 225, 171, 85, 55, 173, 123, 21, 147, 97, 21, 237, 11, 254, 129, 2, 131, 101, 63, 149, 61]
plaintext = [(i, chr(((i * i) % 256) ^ x ^ 0x99)) for i, x in enumerate(ciphertext)]

css = requests.get('https://pay2win.chall.pwnoh.io/main.css').text
index_order = {int(index): int(order) for index, order in re.findall('flag\-char\-(\d+) {\n\s+order: (\d+)', css)}

ordered_plaintext = sorted(plaintext, key=lambda e: index_order[e[0]])
print(''.join(c for _, c in ordered_plaintext))
```
