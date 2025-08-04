
```
nmap -sC -sV -p$(nmap -p- -Pn TARGET | grep "/tcp\|/udp" | cut -d"/" -f1 | tr "\n" ", ") TARGET
```