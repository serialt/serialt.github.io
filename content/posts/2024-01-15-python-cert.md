+++
title = 'Python Cert'
date = 2024-01-16T20:26:27+08:00
draft = false

tags = ["python-cert"]
categories = ["DevOps"]

+++
# Python 自签名证书

mozilla 维护者一个公共ca的文件，在使用http客户端使用自签名证书的时候可以把对应的文件合并然后再导入，示例代码：



`requirements.txt`

```
certifi==2023.11.17
urllib3==2.1.0
```

```python
#!/usr/bin/env python3
# coding=utf-8
# ***********************************************************************
# Description   : Blue Planet
# Author        : serialt
# Created Time  : 2024-01-15 20:12:32
# Last modified : 2024-01-16 20:35:00
# FilePath      : /urlib3/lib.py
# Other         :
#               :
#
#
#
# ***********************************************************************
import os
import urllib3
import certifi

pablic_ca = 'pub_ca.crt'


def load_ca(path):
    if not os.path.exists(pablic_ca):
        mozilla_ca = certifi.where()
        with open(pablic_ca, "a")as pub_ca:
            # get cert from certifi
            m_ca = open(mozilla_ca)
            m_ca_data = m_ca.read()
            pub_ca.write(m_ca_data)
            m_ca.close()

            # get cert from ca_path
            ca_files = os.listdir(path)
            for file in ca_files:
                if not os.path.isdir(file):
                    fl = open(path+'/'+file)
                    data = fl.read()
                    fl.close()
                    pub_ca.write('\n'+data)

        pub_ca.close()
    # Creating a PoolManager instance for sending requests.
    http = urllib3.PoolManager(
        cert_reqs="CERT_REQUIRED",
        maxsize=10,
        ca_certs=pablic_ca,

    )

    # Sending a GET request and getting back response as HTTPResponse object.
    resp = http.request("GET", "http://httpbin.org/get")

    # Print the returned data.
    print(resp.data)


load_ca('ca')

```

