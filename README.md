XSS ChEF - Chrome Extension Exploitation Framework
======

by [Krzysztof Kotowicz](http://blog.kotowicz.net)
v.0.1

About
-----
This is a Chrome Extension Exploitation Framework - think [BeEF](http://beefproject.com/) for Chrome extensions.
Whenever you encounter a XSS vulnerability in Chrome extension, ChEF will ease the exploitation.

What can you actually do (when having appropriate permissions)?
    
  - Monitor open tabs of victims
  - Execute JS on every tab (global XSS)
  - Extract HTML, read/write cookies (also httpOnly), localStorage
  - Get and manipulate browser history
  - Stay persistent until whole browser is closed (or even futher if you can persist in extensions' localStorage)
  - Make screenshot of victims window
  - Further exploit e.g. via attaching BeEF hooks, keyloggers etc.

Installation & usage
------------
### Setup CHeF server (on attacker's machine)

ChEF comes in two different flavours: *PHP/XHR* and *node.js/websocket* version. 
#### PHP 
PHP requires only a PHP and a HTTP server (Apache/nginx) for hosting attacker command & control center, but the communication with hooked browsers has certain latency as it is based on XMLHttpRequest polling.

To install PHP version just download the files somewhere within your document root.
#### Node.js
Node.js version requires a [node.js](http://nodejs.org/) installation and is much faster as it is based on [WebSockets](http://dev.w3.org/html5/websockets/) protocol.

Installation:

    $ npm install websocket
    $ npm install node-static
    $ node server.js [chosen-tcp-port]
    
### Launch CHeF console (on attacker's machine)
  - PHP: http://127.0.0.1/console.php
  - node.js: http://127.0.0.1:8080/

### Hook Chrome extension (on victim's)
First, you have to find a XSS vulnerability in a Google Chrome addon. I won't help you here.
This is similar to looking for XSS in webpages, but totally different, as there are way more DOM based XSSes than reflected ones and the debugging is different.

Once you found a vulnarable extension, inject it with CheF hook script. See 'hook' menu item in console UI for the hook code.

ChEF ships with an exemplary XSS-able chrome addon in `vulnerable_chrome_extension` directory. Install this unpackaged extension (Tools, Extensions, Developer mode, load unpacked extension) in Chrome to test.

### Exploit ###
Once code has been injected and run, a notification should be sent to console, so you can choose the hook by clicking on a 'choose hooked browser' icon on the left and start exploiting.

How does it work?
=================


                   ATTACKER                                VICTIM(S)

                                                                          +------------+
                                                                          |  tab 1     |
                                                                 command  | http://..  |
                                                               +---------->            |
                                                               |          +------------+
                                                               |
       +------------+                              +-----------+-+
       |  console   |                              | addon w/XSS |  result+------------+
       |            |   +-------------+  (XHR/WS)  |             |<------+|  tab 2     |
       |            |+->| ChEF server |<----------+|             |+------>+ https://.. |
       |            |<-+|             |+---------->|  ChEF hook  |        |            |
       |            |   +-------------+            |             |        +------------+
       +------------+                              +-----------+-+
                                                               |
                                                               |          +------------+
                                                               |          |  tab 3     |
                                                               +----------> https://.. |
                                                                          |            |
                                                                          +------------+
                                                                          
Chrome addons usually have permissions to access inidividual tabs in the browser. They can also inject JS code into those tabs. So addons are theoretically cabable of doing a global XSS on any tab. When there is a exploitable XSS vulnerability within a Chrome addon, attacker (with ChEF server) can do exactly that. 

Script injected into Chrome extension (ChEF hook served from a ChEF server) moves to extension background page and installs JS code into every tab it has access to. This JS code listens for various commands from the addon and responds to them. And ChEF-hooked addon receives commands and responds to them by connecting to CHeF server on attackers machine (using XMLHttpRequest or WebSockets connection). Attacker has also a nice web-based UI console to control this whole XSS-based botnet.

