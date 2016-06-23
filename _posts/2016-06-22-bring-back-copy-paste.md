---
layout: news_item
author: cbisnett
categories: [Tips & Tricks]
title: Bring Back Copy-Paste
---

Have you ever tried to copy and paste your password from your password manager (you do use a password manager right?) only to find that the developers removed the ability to paste for _"security reasons"_? First I'm not sure why pasting my password makes me less secure, but I can only assume this was invented to deter users from saving their passwords in `passwords.txt` files. Really this isn't securing those users since they will still use those files and will just choose passwords that are easier to type.

With the increase in popularity of password managers like [LastPass](https://lastpass.com/), [1Password](https://1password.com/), and [KeePass](http://keepass.info/), users, like myself, are making increasingly complex passwords that are *very* annoying to type but also more secure. Sometimes I still find myself needing to copy and past the password because of poor or "interesting" design decisions on behalf of the website.

I keep running into this so I figured I would pass along my usual solutions for getting around this security anti-pattern. Both of these methods require you to use the Chrome Console. To open the console with the textbox already selected, right-click on the control and select `Inspect`. Then switch to the console view or by pressing {% raw %}<span class="kbd">Esc</span>{% endraw %} or clicking the "Console" tab. The following commands assume the offending textbox is still selected.

### Removing the `onPaste()` handler

Often the developer did something simple like setting a handler that gets called for the `paste` event. It can be something as simple as this:`<input type="text" value="" id="myInput" onpaste="return false" />`

The easiest way to remove this is to clear the handler function. Type this command into the console `$0.onpaste = null`

### Removing all event listeners

Sometimes the developer was extra evil and added multiple methods to keep the user from pasting. In these cases I usually just nuke all the event listeners. You can get a list of the registered listeners in the right-hand pane of the console. Each of the entries can be nuked with the above command, just remember that each entry is prefixed with `on`.

If the site uses [jQuery](https://api.jquery.com/) (and who isn't these days ;) then simply `$($0).unbind()` and all event listeners will be gone. I've had this go bad for me before where one of the events was testing the password strength and would only let you submit the form if the password was _good enough_ so you may find that you have to do more hacking ;) or only nuke certain event listeners.

### More evil?

I know there are likely even more evil tricks developers are using. If you come across one, let me know on Twitter [@chrisbisnett](https://twitter.com/chrisbisnett). Maybe one day developers and organizations will realize this practice does not make their users more secure.

### Try it out!

Just try to paste into this textbox, I dare you!
<input type="text" value="" id="myInput" onpaste="return false" />
