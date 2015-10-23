Node.js Authenticator
=====================

Two- and Multi- Factor Authenication (2FA / MFA) for node.js

![](https://blog.authy.com/assets/posts/authenticator.png)

There are a number of apps that various websites use to give you 6-digit codes to increase security when you log in:

* Authy [iPhone](https://itunes.apple.com/us/app/authy/id494168017?mt=8) • [Android](https://play.google.com/store/apps/details?id=com.authy.authy&hl=en) • [Chrome](https://chrome.google.com/webstore/detail/authy/gaedmjdfmmahhbjefcbgaolhhanlaolb?hl=en) • [Linux](https://www.authy.com/personal/) • [OS X](https://www.authy.com/personal/) • [BlackBerry](https://appworld.blackberry.com/webstore/content/38831914/?countrycode=US&lang=en)
* Google Authenticator [iPhone](https://itunes.apple.com/us/app/google-authenticator/id388497605?mt=8) • [Android](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=en)
* Microsoft Authenticator [Windows Phone](https://www.microsoft.com/en-us/store/apps/authenticator/9wzdncrfj3rj) • [Android](https://play.google.com/store/apps/details?id=com.microsoft.msa.authenticator)
* GAuth [FxOS](https://marketplace.firefox.com/app/gauth/)

There are many [Services that Support MFA](http://lifehacker.com/5938565/heres-everywhere-you-should-enable-two-factor-authentication-right-now),
including Google, Microsoft, Facebook, and Digital Ocean for starters.

This module uses [`notp`](https://github.com/guyht/notp) which implements `TOTP` [(RFC 6238)](https://www.ietf.org/rfc/rfc6238.txt)
(the *Authenticator* standard), which is based on `HOTP` [(RFC 4226)](https://www.ietf.org/rfc/rfc4226.txt)
to provide codes that are exactly compatible with all other *Authenticator* apps and services that use them.

Usage
=====

```bash
npm install authenticator --save
```

```javascript
'use strict';

var authenticator = require('authenticator');

var formattedKey = authenticator.generateKey();
// "acqo ua72 d3yf a4e5 uorx ztkh j2xl 3wiz"

var formattedToken = authenticator.generateToken(formattedKey);
// "957 124"

authenticator.verifyToken(formattedKey, formattedToken);
// { delta: 0 }

authenticator.verifyToken(formattedKey, '000 000');
// null
```

### API

### generateKey()

generates a 32-character (160-bit) base32 key

### generateToken(formattedKey)

generates a 6-digit (20-bit) decimal time-based token

### verifyToken(formattedKey, formattedToken)

validates a time-based token within a +/- 30 second (90 seconds) window

returns `null` on failure or an object such as `{ delta: 0 }` on success

QR Code
-------

See <https://davidshimjs.github.io/qrcodejs/> and <https://github.com/soldair/node-qrcode>.

Example use with `qrcode.js` in the browser:

```javascript
'use strict';

var el = document.querySelector('.js-qrcode-canvas');
var link = "otpauth://totp/{{NAME}}?secret={{KEY}}";
var name = "Your Service";
                                              // remove spaces, hyphens, equals, whatever
var key = "acqo ua72 d3yf a4e5 uorx ztkh j2xl 3wiz".replace(/\W/g, '').toLowerCase();

var qr = new QRCode(el, {
  text: link.replace(/{{NAME}}/g, name).replace(/{{KEY}}/g, key)
});
```

Formatting
----------

All non-alphanumeric characters are ignored, so you could just as well use hyphens
or periods or whatever suites your use case.

These are just as valid:

* "acqo ua72 d3yf a4e5 - uorx ztkh j2xl 3wiz"
* "98.24.63"

0, 1, 8, and 9 also not used (so that base32).
To further avoid confusion with O, o, L, l, I, B, and g
you may wish to display lowercase instead of uppercase.

TODO: should this library replace 0 with o, 1 with l (or I?), 8 with b, 9 with g, and so on?

90-second Window
----------------

The window is set to +/- 1, meaning each token is valid for a total of 90 seconds
(-30 seconds, +0 seconds, and +30 seconds)
to account for time drift (which should be very rare for mobile devices)
and humans who are handicapped or otherwise struggle with quick fine motor skills (like my grandma).


Why not SpeakEasy?
------------------

I took a look at the code and I didn't feel comfortable using it.

For any module related to security I want to see that the code is clean,
well-maintained, and that any security-related bugs are addressed.

The author was obviously not well-versed in JavaScript at the time
that he wrote it and it hasn't been cleaned up since.
Also, the author hasn't been responsive to issues and pull requests.

The notp author has been responsive, but notp doesn't do everything I would like.
