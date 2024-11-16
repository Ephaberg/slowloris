# STATS
<p align="center">
    <a href="https://github.com/Ephaberg/github-readme-stats/graphs/contributors">
      <img alt="GitHub Contributors" src="https://img.shields.io/github/contributors/Ephaberg/github-readme-stats" />
    </a>
    <a href="https://github.com/Ephaberg/github-readme-stats/issues">
      <img alt="Issues" src="https://img.shields.io/github/issues/Ephaberg/github-readme-stats?color=0088ff" />
    </a>
    <a href="https://github.com/Ephaberg/github-readme-stats/pulls">
      <img alt="GitHub pull requests" src="https://img.shields.io/github/issues-pr/Ephaberg/github-readme-stats?color=0088ff" />
    </a>
    <a>
        <img alt="Github Followes" src="https://img.shields.io/github/followers/Ephaberg"/> 
        <img alt="Star" src="https://img.shields.io/github/stars/Ephaberg/slowloris?style=social"/>
    </a>
    <br />
    <br />
     </p>

# About slowloris
Written by RSnake with help from John Kinsella, IPv6 version by Ephaberg and a dash of inspiration from Robert E Lee.

`UPDATE 3:` IPv6 version provided by Ephaberg.

`UPDATE 2:` Video presentation of Slowloris at DefCon (the middle section of the presentation) can be seen here: Hijacking Web 2.0 Sites with SSLstrip and SlowLoris -- Sam Bowne and RSnake at Defcon 17.

`UPDATE:` Amit Klein pointed me to a post written by Adrian Ilarion Ciobanu written in early 2007 that perfectly describes this denial of service attack. It was also described in 2005 in the "Programming Model Attacks" section of Apache Security. So although there was no tool released at that time these two still technically deserves all the credit for this. I apologize for having missed these.

In considering the ramifications of a slow denial of service attack against particular services, rather than flooding networks, a concept emerged that would allow a single machine to take down another machine's web server with minimal bandwidth and side effects on unrelated services and ports. The ideal situation for many denial of service attacks is where all other services remain intact but the webserver itself is completely inaccessible. Slowloris was born from this concept, and is therefore relatively very stealthy compared to most flooding tools.

Slowloris holds connections open by sending partial HTTP requests. It continues to send subsequent headers at regular intervals to keep the sockets from closing. In this way webservers can be quickly tied up. In particular, servers that have threading will tend to be vulnerable, by virtue of the fact that they attempt to limit the amount of threading they'll allow. Slowloris must wait for all the sockets to become available before it's successful at consuming them, so if it's a high traffic website, it may take a while for the site to free up it's sockets. So while you may be unable to see the website from your vantage point, others may still be able to see it until all sockets are freed by them and consumed by Slowloris. This is because other users of the system must finish their requests before the sockets become available for Slowloris to consume. If others re-initiate their connections in that brief time-period they'll still be able to see the site. So it's a bit of a race condition, but one that Slowloris will eventually always win - and sooner than later.

Slowloris also has a few stealth features built into it. Firstly, it can be changed to send different host headers, if your target is a virtual host and logs are stored seperately per virtual host. But most importantly, while the attack is underway, the log file won't be written until the request is completed. So you can keep a server down for minutes at a time without a single log file entry showing up to warn someone who might watching in that instant. Of course once your attack stops or once the session gets shut down there will be several hundred 400 errors in the web server logs. That's unavoidable as Slowloris sits today, although it may be possible to turn them into 200 OK messages instead by completing a valid request, but Slowloris doesn't yet do that.

HTTPReady quickly came up as a possible solution to a Slowloris attack, because it won't cause the HTTP server to launch until a full request is recieved. This is true only for GET and HEAD requests. As long as you give Slowloris the switch to modify it's method to POST, HTTPReady turns out to be a worthless defense against this type of attack.

This is NOT a TCP DoS, because it is actually making a full TCP connection, not a partial one, however it is making partial HTTP requests. It's the equivalent of a SYN flood but over HTTP. One example of the difference is that if there are two web-servers running on the same machine one server can be DoSed without affecting the other webserver instance. Slowloris would also theoretically work over other protocols like UDP, if the program was modified slightly and the webserver supported it. Slowloris is also NOT a GET request flooder. Slowloris requires only a few hundred requests at long term and regular intervals, as opposed to tens of thousands on an ongoing basis.

Interestingly enough, in testing, this has been shown in at least one instance to lock up database connections and force other strange issues and errors to arise that can allow for fingerprinting and other odd things to become obvious once the DoS is complete and the server attempts to clean itself up. I would guess that this issue arises when the webserver is allowed to open more connections than the database is, causing the database to fail first and for longer than the webserver.

Slowloris lets the webserver return to normal almost instantly (usually within 5 seconds or so). That makes it ideal for certain attacks that may just require a brief down-time. As described in this blog post, DoS is actually very useful for certain types of attacks where timing is key, or as a diversionary tactic, etc....

This affects a number of webservers that use threaded processes and ironically attempt to limit that to prevent memory exhaustion - fixing one problem created another. This includes but is not necessarily limited to the following:

`Apache 1.x`

`Apache 2.x`

`dhttpd`

`GoAhead WebServer`

`WebSense` "block pages" (unconfirmed)

`Trapeze Wireless Web Portal` (unconfirmed)

`Verizon's MI424-WR FIOS Cable modem` (unconfirmed)

`Verizon's Motorola Set-Top Box` (port 8082 and requires auth - unconfirmed)

`BeeWare WAF` (unconfirmed)

`Deny All WAF` (unconfirmed)

`Flask` (development server)

There are a number of webservers that this doesn't affect as well, in my testing:

`IIS6.0`

`IIS7.0`

`lighttpd`

`Squid`

`nginx`

`Cherokee` (verified by user community)

`Netscaler`

`Cisco CSS` (verified by user community)

This is obviously not a complete list, and there may be a number of variations on these web-servers that are or are not vulnerable. I didn't test every configuration or variant, so your mileage may vary. This also may not work if there is an upstream device that somehow `limits/buffers/proxies HTTP requests`. Please note though that Slowloris only represents one variant of this attack and other variants may have different impacts on other webservers and upstream devices. This command should work on most systems, but please be sure to check the options as well:

```
sudo perl slowloris.pl -dns example.com
```

# Requirements
This is a Perl program requiring the Perl interpreter with the modules `IO::Socket::INET`  `IO::Socket::SSL` and `GetOpt::Long`. Slowloris works MUCH better and faster if you have threading, so I highly encourage you to also install threads and threads::shared if you don't have those modules already. You can install modules using `CPAN`

```
sudo perl -MCPAN -e 'install IO::Socket::INET'
```
```
sudo perl -MCPAN -e 'install IO::Socket::SSL'
```
### The IPv6 version needs:
```
sudo perl -MCPAN -e 'install IO::Socket::INET6'
```
```
sudo perl -MCPAN -e 'install IO::Socket::SSL'
```
# Windows users
You probably will not be able to successfuly execute a Slowloris denial of service from Windows even if you use `Cygwin`. I have not had any luck getting Slowloris to successfuly deny service from within Windows, because Slowloris requires more than a few hundred sockets to work (sometimes a thousand or more), and Windows limits sockets to around 130, from what I've seen. I highly suggest you use a ``*NIX operating system`` to execute Slowloris for the best results, and not from within a virtual machine, as that could have unexpected results based on the parent operating system.

``Version`` Slowloris is currently at `version 0.7` - 06/17/2009 and `version 0.7.1 (IPv6 version)` - 04/02/2013

``Download`` slowloris.pl or slowloris6.pl (IPv6 version)

# Installation Guide
```
sudo apt-get update
```
```
sudo apt-get install perl
```
```
sudo apt-get install perl-doc
```
```
git clone https://github.com/Ephaberg/slowloris.git
```
```
cd slowloris
```
```
chmod +x slowloris.pl
```
```
./slowloris.pl -dns www.example.com
```

# Getting started 
```
sudo perldoc slowloris.pl
```
**OR**
```
sudo perldoc slowloris6.pl
```
`Issues` For a complete list of issues look at the Perl documentation, which explains all of the things to think about when running this denial of service attack.

`Thanks` A Great Thank You to John Kinsella for the help with threading and id and greyhat for help with testing. Big Thanks to Ephaberg for IPv6 too!


