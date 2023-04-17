---
title: "Malicious Document Analysis"
subtitle: "Analyzing Office Documents"
date: 2023-04-17T16:21:50-03:00
draft: false
author: "HollowSec"
authorLink: "https://twitter.com/hollowsec_"
authorEmail: ""
description: ""
keywords: "malware,office document, VBA Macro, analysis"
license: ""
comment: false
weight: 0

tags:
- analysis
- malware
- office document
- VBA Macro
categories:
- analysis
- malware
- VBA Macro

hiddenFromHomePage: false
hiddenFromSearch: false

summary: ""
resources:
- name: featured-image
  src: featured-image.jpg
- name: featured-image-preview
  src: featured-image-preview.jpg

toc:
  enable: true
math:
  enable: false
lightgallery: false
seo:
  images: []

repost:
  enable: true
  url: ""

# See details front matter: https://fixit.lruihao.cn/theme-documentation-content/#front-matter
---

We are going to be using VirusTotal and command line tools like Olevba
``` bash
sudo -H pip install -U oletools
```

MD5 Hash of the file we gonna analyze
```
a3b613d128aace09241504e8acc678c2
```

First of all we can throw the file on Virus Total to see what we can get

![virustotal1.png](/img/virustotal1.png)



On Virus Total we can see the behavior of the malware before we execute, and we see all the security vendors that are flagging the file as malicious.

Also, we can see more details about the file
![virustotal2.png](/img/virustotal2.png)



On the Relations tab, we can see the contacted IP Addresses
![virustotal3.png](/img/virustotal3.png)
This means, when we execute the file, he tries to connect these IP Addresses.


Now let's go to the terminal and start our advanced static analysis
First of all we gonna start using olemeta to see the metadata of the file
```
olemeta blog.doc
```
![olemeta.png](/img/olemeta.png)
On the template field we can see the value 'Normal.dotm'. 'm' meaning macro

Next we gonna use oleid

```
oleid blog.doc
```

![oleid.png](/img/oleid.png)
oleid tells us that the file has VBA Macros and is suspicious with risk High. On Description tell us to use olevba to analyse

```
olevba blog.doc
```

![olevba.png](/img/olevba.png)
The output give us indicators that are in the VBA macros and highlight the suspicious keywords for us
IOC = Indicators of compromise.

Searching for these IP's we can found:
!![IP.png](/img/IP.png)

script trying to ping the IP 1.1.2.2
Now we can try decode all the malicious VBA Macro script

```bash
olevba blog.doc > blog.vba

olevba --deobf --reveal blog.vba > blog-deobf.vba
```

Using Visual Code we can open the file

![vcode.png](/img/vcode.png)

After decoding we can see much more information

![decode.png](/img/decode.png)


Looking at the code we can see a part of a url
![url.png](/img/url.png)
If we search for 'JASHDUIQWHDKJQAD' in the code we find:

![string.png](/img/string.png)

We can substitue the 'JASHDUIQWHDKJQAD'  for '.44/upd/install' in the entire code

After that we have the following code:

![code.png](/img/code.png)

```
"strRT = ""http://91.220.131" + .44/upd/install + ".exe"""
```

http[:]//91.220.131.44/upd/install.exe


Now we can do the same for the following piece
![temp-obf.png](/img/temp-obf.png)

![temp.png](/img/temp.png)

We found where is gonna save the file

'c:\Windows\Temp\444.exe'

Now we can clearly see what the malicious VBA Macro Script is doing

![script.png](/img/script.png)

It's trying to download a .exe file using the command 'New-Object System.Net.WebClient' and store it on the Temp directory with the name 444.exe



# That's All Folks
