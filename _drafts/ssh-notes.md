
## Create a local SSH tunnel

```
$ ssh -N -L 192.168.0.39:8000:192.168.1.6:80 user@serverone.mydomain.com
```

## Create a remote SSH tunnel

```
$ ssh -N -R serverone.mydomain.com:8888:192.168.0.39:80 user@serverone.mydomain.com
```

## References
- https://linuxconfig.org/introduction-to-ssh-port-forwarding
