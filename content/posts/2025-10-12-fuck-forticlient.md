+++

title = 'un-forticlient'
date = 2025-10-12T22:26:27+08:00
draft = false

tags = ["forticlient"]
categories = ["VPN"]

+++
`卸载forticlient`



```
/bin/ls -dleO@ /Applications/FortiClient.app
sudo /usr/bin/chflags -R noschg /Applications/FortiClient.app
```

然后就可以删除