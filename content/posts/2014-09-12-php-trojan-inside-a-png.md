---
title: PHP trojan inside a PNG
author: Jesús Sánchez
type: post
date: 2014-09-12T02:32:14+00:00
url: /php-trojan-inside-a-png/
categories:
  - Tecnología

---
During a recent security audit I found a PHP script obfuscated inside a filename called social.png, that was used via an include call from another PHP file. This script came inside a pirated version of a commercial plugin for WordPress. The file is identified as **Trojan.PHP.Shell.W** or **PHP/Alter.A** by some malware checking tools (ESET-NOD32, BitDefender).

Inside the script I decoded the following data: domain names, emails and a public key. They are being used for malicious purposes (spread malware) mostly on WordPress.

  * Known [malicious domain names][1].
  * Known [malicious email addresses][2].
  * Known [malicious public keys][3].

Once your server has been compromised it sends a message to ALL the email addresses informing the attacker(s) that they have shell access to your machine.

Luckily for my client his server is configured to send emails only by authenticated clients, so my client was never exposed.

### Suggestions

  * Check your logs for any activity on these domains and addresses, or better yet, block them on your server&#8217;s firewall if possible.
  * Check you WordPress database inside the (wp)\_options table for the field WP\_CLIENT_KEY, remove it if it matches the malicious public key posted above or if you don&#8217;t use an external admin panel for your site.
  * Avoid installing plugins from untrustworthy sources (pirate sites, nulled scripts, et al.)

 [1]: https://blog.jesvs.com/wp-content/uploads/2017/11/malicious-domains.txt
 [2]: https://blog.jesvs.com/wp-content/uploads/2017/11/malicious-emails.txt
 [3]: https://blog.jesvs.com/wp-content/uploads/2017/11/malicious-keys.txt