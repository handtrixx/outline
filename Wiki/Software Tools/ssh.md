# ssh

## Client Config

in the users directory in subfolder ".ssh" edit/create file "config"

```bash
Host *
    ServerAliveInterval 30
    ServerAliveCountMax 3 
    TCPKeepAlive no
    
Host NAME_USED_FOR_OPENING_CONNECTION
  HostName IP
  Port PORT
  User USERNAME

...
```