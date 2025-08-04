
FTP has 2 modes:
	Active - client tells server which port for data - server starts that TCP session
	Passive - server advertises data port - client starts that TCP session

[[FTP Commands]]

[[FTP Return Codes]]

- Always check for Anonymous user - no creds needed
	-  ```ftp a.b.c.d``` 
	- username: anonymous
	- password: if needed - can be whatever

- Download all files:
	```wget-all-files
	wget -m --no-passive ftp://anonymous:anonymous@<target-ftp-ip-address>
	```
- Use ncat or telnet:
	```ftp-nc-telnet
	nc -nv <target-ip> 21
	telnet <target-ip> 21
	```
- Use openssl is FTP runs with TLS/SSL:
	```ftp-openssl-client
	openssl s_client -connect 10.129.14.136:21 -starttls ftp
	```

