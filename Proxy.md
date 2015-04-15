Some Squid proxy servers use HTTP between client and proxy server for both HTTP and HTTPS requests to the destination. This causes problems for the GData Python API, which doesn't seem to handle it.

I've filed an issue: https://code.google.com/p/gdata-python-client/issues/detail?id=618&thanks=618&ts=1337610710

I've also created a clone of the GData library that has a patch applied to fix the issue: https://code.google.com/r/jimlawton-proxyfix/

To use the patched clone:
```
 # hg clone https://jim.lawton@code.google.com/r/jimlawton-proxyfix/ 
 # cd jimlawton-proxyfix/
 # sudo python setup.py install -f
```