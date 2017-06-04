php-mail-signature
==================

A standalone PHP class to sign your e-mails with DKIM and Domain Keys. License LGPL v2.1

Project page : https://github.com/louisameline/php-mail-signature

This class is based on the work made on PHP-MAILER with the following differences :
- it is a standalone class, easy to use
- it supports the Domain Keys header
- it supports UTF-8
- it will let you choose the headers you want to base the signature on
- it will let you choose between simple and relaxed body canonicalization

If the class fails to sign the e-mail, the returned DKIM header will be empty and the mail will still be sent, just unsigned. A php warning is thrown for logging.

**NOTE**: you will NOT be able to use Domain Keys with PHP's mail() function, since it does not allow to prepend the DK header before the To and Subject ones. DKIM is ok with that, but Domain Keys is not. If you still want Domain Keys, you will have to manage to send your mail straight to your MTA without the default mail() function.

**NOTE 2**: if your MTA is Qmail, make sure to read the note at the end of the Example below.

Successfully tested against Gmail, Yahoo Mail, Live.com, AOL.com, appmaildev.com.
I hope it helps and saves you plenty of time. Feedback is welcome.

For more info, you should read http://www.ietf.org/rfc/rfc4871.txt and http://www.zytrax.com/books/dns/ch9/dkim.html

```php
<?php

// 0) YOUR E-MAIL

$to = 'test@example.com';
$subject = 'My subject';

$headers =
	'MIME-Version: 1.0
	From: "Sender" <sender@example.com>
	Content-type: text/html; charset=utf8';

$message =
	'<html>
		<header></header>
		<body>
			Hello, this a DKIM test e-mail
		</body>
	</html>';



// 1) YOU USUALLY DID :
mail($to, $subject, $message, $headers);



// 2) NOW YOU WILL DO (after setting up the config file and your DNS records) :

// Make sure linefeeds are in CRLF format - it is essential for signing
$message = preg_replace('/(?<!\r)\n/', "\r\n", $message);
$headers = preg_replace('/(?<!\r)\n/', "\r\n", $headers);

require_once 'mail-signature.class.php';
require_once 'mail-signature.config.php';

$signature = new mail_signature(
	MAIL_RSA_PRIV,
	MAIL_RSA_PASSPHRASE,
	MAIL_DOMAIN,
	MAIL_SELECTOR
);
$signed_headers = $signature -> get_signed_headers($to, $subject, $message, $headers);

mail($to, $subject, $message, $signed_headers.$headers);



// 3) OR USE OPTIONS TO ADD SOME FLAVOR :

$message = preg_replace('/(?<!\r)\n/', "\r\n", $message);
$headers = preg_replace('/(?<!\r)\n/', "\r\n", $headers);

$options = array(
	'use_dkim' => false,
	'use_domainKeys' => true,
	'identity' => MAIL_IDENTITY,
	// if you prefer simple canonicalization (though the default "relaxed"
	// is recommended)
	'dkim_body_canonicalization' => 'simple',
	'dk_canonicalization' => 'nofws',
	// if you want to sign the mail on a different list of headers than the
	// default one (see class constructor). Case-insensitive.
	'signed_headers' => array(
		'message-Id',
		'Content-type',
		'To',
		'subject'
	)
);

require_once 'mail-signature.class.php';
require_once 'mail-signature.config.php';

$signature = new mail_signature(
	MAIL_RSA_PRIV,
	MAIL_RSA_PASSPHRASE,
	MAIL_DOMAIN,
	MAIL_SELECTOR,
	$options
);
$signed_headers = $signature -> get_signed_headers($to, $subject, $message, $headers);

mail($to, $subject, $message, $signed_headers.$headers);



// NOTE: Qmail requires that the message and headers you pass to it have `\n` line breaks
// (if you find other MTAs that have this behaviour, please let me know). To make things
// work, just add these three lines right before your call to `mail(...)`:

$message = str_replace("\r\n", "\n", $message);
$headers = str_replace("\r\n", "\n", $headers);
$signed_headers = str_replace("\r\n", "\n", $signed_headers);
```
