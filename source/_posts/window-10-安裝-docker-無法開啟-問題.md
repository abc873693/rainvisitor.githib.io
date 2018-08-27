---
title: window 10 安裝 docker 無法開啟 問題
comments: true
categories:
  - Docker
abbrlink: b0c060c4
date: 2018-08-27 16:21:28
tags:
---

若開啟docker後，vmware無法執行須關閉docker使用的hyper-v兩者會相衝，只能同時開其一
https://stackoverflow.com/questions/48066994/docker-no-matching-manifest-for-windows-amd64-in-the-manifest-list-entries


關閉系統hyper-v
{% codeblock lang:bat %}
    bcdedit /set hypervisorlaunchtype off
{% endcodeblock %}

重新打開
{% codeblock lang:bat %}
    bcdedit /set hypervisorlaunchtype auto
{% endcodeblock %}

上述兩指令 執行完都須重新開機