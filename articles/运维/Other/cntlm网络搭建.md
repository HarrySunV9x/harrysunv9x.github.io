---
title: cntlm网络搭建
data: 2024-05-17
---

配置cntlm.ini:

Username, Domain, Password（通过密文则不需要）, Proxy, noProxy, Listen

得到密文：

cntlm -H -u username，输入密码，得到密文一共有三个：PassLM、PassNT、PassNTLMv2