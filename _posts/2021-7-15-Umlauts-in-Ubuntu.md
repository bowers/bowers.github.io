---
layout: post
title: Typing umlaut characters using Unicode codes in Ubuntu
---
Use ctrl+shift+u then these character codes to get ä ö ü and ß

If you use a US keyboard need to regularly type non-US characters (like umlauts), the right way is through using a Compose key, [which you can configure using a tool like tweaks](https://help.ubuntu.com/stable/ubuntu-help/tips-specialchars.html.en).

But if you're lazy (like me), and only need to type an occasoinal thing in German, this works for Ubuntu (at least in 20.04+):

press and release CTRL+SHIFT+u.  then type the two-character code below, then press SPACE.

| character | code |
| --------- | ---- |
| ä | e4 |
| ö | f6 |
| ü | fc |
| Ä | c4 |
| Ö | d6 |
| Ü | dc |
| ß | df |

Example: CTRL+SHIFT+U, e4, SPACE.  That gives you ä.
