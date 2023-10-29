# unwyze - a Wyze Cam v3 RCE Exploit

## background story

I worked on auditing the [Wyze Cam V3](https://www.wyze.com/products/wyze-cam) firmware as part of entering this year Pwn2Own 2023 Toronto competition. My entry
came along nicely and I was able to identify and exploit some critical vulnerabilities.

The night before my flight to Toronto I became aware Wyze had just released a firmware update (`4.36.11.7071`) which has the following changelog:

- **Security improvements**

Yeah, that's it; the full changelog/revision history. This update killed an important vulnerability I was relying on: a authentication bypass for the encapsulated DTLS connection to a wyze camera. My entry (and the entry of many others) was killed right there.

I guess Wyze's rationale was they wanted to prevent some kind of mini PR nightmare in the hopes of their camera not getting pwned during pwn2own. That didn't work, some other teams (synacktiv et al.) had additional bugs up their sleeves that did not require the authentication bypass.

But you can't help but wonder why they sat on this patch for so long.. leaving their valued customers vulnerable in the meanwhile!

**UPDATE**: A WyzeCam representative has informed me that:

> I want to clarify a few things; we didn't know about this issue for years, this is an issue in the third-party library we use and we got a report about it just a few days before pwn2own and once we got the report in our bugbounty program we patched the issue in 3 days and released to public.

To celebrate their very wyze (huhu) decision I have decided to release my exploit to the public. Maybe next time they will prefer doing a coordinated disclosure through the ZDI program rather than frustrating a few contestants post-deadline.

## the bugs

### DTLS authentication bypass

Wyze has a daemon (iCamera) that listens on UDP port 32761 speaking some derivative of the [TUTK protocol](https://www.throughtek.com/p2p-iot-connection/). The outer layer of the protocol consists out of scrambled/XOR'd frames using a funny constant (shout out to Charlie; the engineer!). Inside of this custom framing format you can establish a DTLS session with the camera. The only supported ciphersuite is `ECDHE-PSK-CHACHA20-POLY1305` and a typical attacker does not have access to the (device unique) PSK. However there was a fallback method where you could specify a PSK identity that starts with 'AUTHTKN\_' during the TLS handshake in order to be able to pick an arbitrarily chosen PSK.

### Stack buffer overflow in JSON unpacking

Shortly after establishing an authenticated DTLS session with the camera the client sends a packet containing a JSON object blob with a property called `cameraInfo`. Inside this object there is an array with numbers called `audioEncoderList`. The iCamera code responsible for parsing this JSON object will loop over all `audioEncoderList` entries and copy them to a fixed-sized array of integers on the stack.

Of course, since it is 2023 and this is IoT nonsense we shouldn't expect them to have compiled the binary with stack canaries or even as a position independent executable.

Thus we don't need any additional information leaks to bypass ASLR or leak a canary value and we can ROP our way to victory!

## Exploit

The exploit will use the vulnerabilities described above to spawn an interactive (connectback)shell. I have taken the liberty to backport the exploit to some older Wyze cam V3 versions as well, just because.

The exploit has been tested on the following firmwares:

- v4.36.10.4054
- v4.36.11.4679
- v4.36.11.5859

## Closing words

As usual, enjoy the codes and don't ask for support unless there is any good incentive for me to help you out.

To the vendor (Wyze): I hope you will reconsider your ways!

Greets fly out to all old & new friends I met during my stay in Toronto. Many interesting chats were had and many beverages got consumed; good times!

-- blasty `<peter@haxx.in>`
