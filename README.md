# anfd

anfd (Application Networking Firewall Daemon) is a building block for an application firewall for linux. On linux three main components help protecting against attacks of viruses, malware and alike: 
 * A strict separation of privileged users (like root and other system users) and regular user accounts, combined with appropriate restrictions of access rights. Thereby malicious code should be unable to escalate privileges out of any account and infect the system as a whole. 
 * A properly configured iptables firewall allows access from the system to the outside, and, if needed, access from outside to selected services.
 * Most importantly, installing latest security patches on a regular basis, best automatically, keeps the system secure against attacks of any kind, at least to the level to which patches are available.
Still, there is a remaining risk that code contains still unpatched security holes that malicious code might explore, you might by accident download and install harmful code that tries to download further malicious code or leak sensitve information and key material to untrusted parties. 
_anfd_ helps to restrict connections to the outside world not only to specific ip addresses and ports but also to specific applications that are allowed to estaplish connections to these locations. For example you would only expect your email client or mta to establish smtp connections, only a few applications like the web browser, the update mechanism and a hand full command line applications are supposed to download software.

-------------
Configuration
-------------

_anfd_ inspects network traffic that is queued by the regular iptables firewall and accepts or rejects it according to its own rules. These rules are configured in `/etc/anfd.conf`. 

Each line contains an absolute path to an application and a space separated list of locations they are allowed to access. Each location is a fully-qualified hostname or an ip address or subnet in cidr notation, appended by a colon(:) and a port list. The port list consists of the allowed ports as integer numbers, separated by commata(,) or port ranges where lower port and upper port are separated by a dash(-), or a star(*) which denotes any port on that host or network. The star can also be used to mask the hostname and restrict access to a particular domain or subdomain, and allow access to any host below that domain, or if only a star is used as hostname access to all hosts on the sepcified ports for a specific application (note that still other applications may have different restrictions: the web browser may access all pages via http/s and other applications might need access only to specific hosts for fetching updates). If the colon and the port list is ommitted, the rule refers to all ports on the specific network or host.

Everything between a hash sign(#) and the end of a line is regarded as a comment.

You may define and use variables. Variables may contain alphanumeric characters. A definition consists of that name left of an equal sign(=) and the assigned value to the right of it. On subsequent lines you may insert these variable by preceding the variable name with a dollar sign($).

You can disallow access to specific subnets by prefixing it by an exclamation mark(!). By doing so, you can disallow access to hosts within networks to which access was granted previously.

If the name of the application starts with an ampersand sign(&) the rest is interpreted as a regular expression. Note that it is always better to use the absolute path to undoubtedly tie the rule to a specific application.

If the name of the application starts with a circumflex(^), the name is matched as a command line argument of the application. This is useful to match scripts that are all use the same interpreter.

Examples:

* Defining a variable, e.g. for a network
`HOME_NETWORK=192.168.1.0/24`
or the same for a port list, e.g. for email:
`MAIL=465,993,143,25,110,587`
Port ranges are also allowed, like `1024-65536`

* A comment
`# this is a comment`

* Allowing access to subdomains on a specific port e.g. for updates
`/usr/lib/apt/methods/http *.ubuntu.com:80`

* Using a port list:
`/usr/lib/firefox/firefox *:80,443`

* Space separated individual expressions are also fine
`/usr/bin/lynx *:80 *:443`

* Allow access to a specific port and specific host:
`/usr/lib/git-core/git-remote-http github.com:443`

* Allow access to all ports for an application that requires access to many ports and hosts (you might want to restrict that further, but you know this is just an example):
`/usr/bin/skype *`

* A more complex rule may involve a couple of hosts and ports (note that $ADDONS contains a list of hosts and ports where the installed addons get their updates).
`/usr/lib/thunderbird/thunderbird $HOSTER1:$MAIL $HOSTER2:$MAIL $HOSTER3:$MAIL $ADDONS`

* Allow full access to the home network (compare above):
`&.* $HOME_NETWORK`



-------------
Prerequesites
-------------

_anfd_ uses perl and a couple of modules: libnet-cidr-perl libnet-rawip-perl libnfqueue-perl libnetpacket-perl


----------
Installing
----------

1.   copy the files to `/etc` and `/usr/local`

2.   install the prerequisites

3.   enable the anfd service:
     ```
     update-rc.d anfd defaults
     update-rc.d anfd enable
     systemctl enable anfd
     ```

4.   install and enable a firewall, e.g. shorewall

5.   configure rules that make use of NFQUEUE instead of ACCEPT


---------
Arguments
---------

	-h, -?      : Print this help message.
	-D          : Debug mode - don't detach from terminal and print detailed infos
	-i 'command': Use this iptables command to insert the QUEUE rule.
	-x          : Dont insert any iptables rule. Admin will take care of that
	              herself.
	-c file     : Use this config file. 
	-p pidfile  : Use this pid file.
	-k          : Kill running anfd process.


-------
Credits
-------

Originally, anfd as developed by ostcar and otzenpunk was called 
"ain't no firewall daemon" (http://wiki.UbuntuUsers.de/Skripte/anfd) 
By inverting the logic of the ip expressions however, it becomes to a building block
for an application firewall similar as windows users are used to.


----
Bugs
----

 * not really a bug, but IPv6 support is still missing

