Title: Things to think about when implementing SSL/TLS on a public facing web server
Blurb: Testing out support for TLS 1.2
Tags:TLS

##TLDR: you can not have TLS 1.2 and a commercial CA ECDHE certificate yet if you want to have a public facing website.

Some things to think about when implementing site wide SSL/TLS. Remember that this was written in the early part of April 2014.

1. [Signing authorities vs self signed](#signingauthorities)
2. [Server support](#serversupport)
3. [Browser support](#browsersupport)

<h3 id="signingauthorities">Signing authorities vs self signed</h3>
If you go self signed than you can do whatever you want and you will be happy. However if you want to use a CA you are stuck with RSA. At the time of writing I couldn't find a CA that offered to sign a ECDSA certificate (at least for small commercial prices) although I image that it will come eventually.
If you try to have the CA's system sign the certificate you just get an error, some more helpful than others.

<h3 id="serversupport">Server support</h3>
Apache support for ECDHE suites only landed in 2.3. Arch Linux rolled onto 2.4 in March but that is not very fun in a production setup. Debian is still on 2.2 and will go to 2.4 in Jessie (the current testing branch) aka Debian 8.0. This means a typical Debian system will not support ECDHE suites and PFS until probably late this year or even later.

nginx supports ECDHE suites as of now.

Windows, I have no idea (or particularly care to).

CORS filters need to be updated to put an 'https' at the front. Most tutorials et. al. do not mention this as it is not assumed to be plain http.

<h3 id="browsersupport">Browser support</h3>
Firefox just got public release support for TLS 1.2 in the 27.0 release. Chromium in the 30.0 release. Internet Explorer has it on version 11. Safari version 7 on OS X 10.9.

The one that might really hit you however is the search bots do not yet support TLS 1.2. GoogleBot (Oct 1013) only supports SSL 3.0 and TLS 1.0. If you do not support either of them then Google cant index your site (inc webmaster tools). If that is important to you then you might what to think about that.

Additionally, mixed content policy now means that if a secured page tries to lead an 'unsecured' page (i.e. https:// loading something with the uri to http://) then the http:// element wont be loaded. this includes images, video, audio, scripts, fonts, anything. These can even be inside elements that are otherwise secured. For example I use Goggle web fonts extensively in SVGs around this site. The default embedding code is <code>&lt;link href='http://fonts.googleapis.com/css?family=FontYouWant' rel='stylesheet' type='text/css'&gt;</code> which is unsecured. So change all of them to https:// and they load again. N.B. the link that that goes to has unsecured content linked in it which is OK and loads fine (not sure if that should happen or not).


