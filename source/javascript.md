```javascript
// transform all request to include basic auth.
var tonce = new Date().getTime() * 1000; // any tonce
var shaObj = new jsSHA(tonce + publicKey + data, "TEXT");
var hmacSignature = shaObj.getHMAC(privateKey, "TEXT", "SHA-512", "B64");

headers.Authorization = "Basic " + Base64.encode(publicKey + ":" + hmacSignature);
headers.Tonce = tonce;
```