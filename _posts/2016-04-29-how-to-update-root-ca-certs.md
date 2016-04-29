---
title: How To - Update Server's Root CA Certificate's
---

I have seen a uptick the closer we get to June for customers requesting compliance for [PayPal's SSL Certificate Upgrade](https://www.paypal-knowledge.com/infocenter/index?page=content&widgetview=true&id=FAQ1766){:target="_blank"}. If you interact with API's it is a good thing to update the root ca certs, especially on older distros. In this case I am going to show you how to verify support for VeriSign G5 Root Certificate and SHA-256 support.

For Red Hat Enterprise Linux 5, CentOS 5, Fedora, Scientific Linux and all other RHEL 5 based distro's. We are going to download the ca-bundle from the cURL site.

{% highlight text %}
box:/# cp /etc/pki/tls/certs/ca-bundle.crt /etc/pki/tls/certs/ca-bundle.crt.bak

box:/# curl http://curl.haxx.se/ca/cacert.pem -o /etc/pki/tls/certs/ca-bundle.crt

box:/# chmod 444 /etc/pki/tls/certs/ca-bundle.crt
{% endhighlight %}

For Red Hat Enterprise Linux 6 and 7, CentOS 6 and 7, Fedora, Scientific Linux and all other RHEL 6 and 7 based distro's. Here we are going to use the built-in package "ca-certificates"

{% highlight text %}
box:/# update-ca-trust check

box:/# update-ca-trust enable
{% endhighlight %}

For Debian, Ubunutu and other distro's based on Debian. I have [included the certificate](https://knowledge.verisign.com/support/mpki-for-ssl-support/index?page=content&actp=CROSSLINK&id=SO5624){:target="_blank"} which you will echo into /usr/local/share/ and then use update-ca-certificates to activate it. You may need to install the update-ca-certificates package if it is not already installed.

{% highlight text %}
cat > /usr/local/share/ca-certificates/verisigng5.crt << '_EOF'
-----BEGIN CERTIFICATE-----
MIIE0zCCA7ugAwIBAgIQGNrRniZ96LtKIVjNzGs7SjANBgkqhkiG9w0BAQUFADCB
yjELMAkGA1UEBhMCVVMxFzAVBgNVBAoTDlZlcmlTaWduLCBJbmMuMR8wHQYDVQQL
ExZWZXJpU2lnbiBUcnVzdCBOZXR3b3JrMTowOAYDVQQLEzEoYykgMjAwNiBWZXJp
U2lnbiwgSW5jLiAtIEZvciBhdXRob3JpemVkIHVzZSBvbmx5MUUwQwYDVQQDEzxW
ZXJpU2lnbiBDbGFzcyAzIFB1YmxpYyBQcmltYXJ5IENlcnRpZmljYXRpb24gQXV0
aG9yaXR5IC0gRzUwHhcNMDYxMTA4MDAwMDAwWhcNMzYwNzE2MjM1OTU5WjCByjEL
MAkGA1UEBhMCVVMxFzAVBgNVBAoTDlZlcmlTaWduLCBJbmMuMR8wHQYDVQQLExZW
ZXJpU2lnbiBUcnVzdCBOZXR3b3JrMTowOAYDVQQLEzEoYykgMjAwNiBWZXJpU2ln
biwgSW5jLiAtIEZvciBhdXRob3JpemVkIHVzZSBvbmx5MUUwQwYDVQQDEzxWZXJp
U2lnbiBDbGFzcyAzIFB1YmxpYyBQcmltYXJ5IENlcnRpZmljYXRpb24gQXV0aG9y
aXR5IC0gRzUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCvJAgIKXo1
nmAMqudLO07cfLw8RRy7K+D+KQL5VwijZIUVJ/XxrcgxiV0i6CqqpkKzj/i5Vbex
t0uz/o9+B1fs70PbZmIVYc9gDaTY3vjgw2IIPVQT60nKWVSFJuUrjxuf6/WhkcIz
SdhDY2pSS9KP6HBRTdGJaXvHcPaz3BJ023tdS1bTlr8Vd6Gw9KIl8q8ckmcY5fQG
BO+QueQA5N06tRn/Arr0PO7gi+s3i+z016zy9vA9r911kTMZHRxAy3QkGSGT2RT+
rCpSx4/VBEnkjWNHiDxpg8v+R70rfk/Fla4OndTRQ8Bnc+MUCH7lP59zuDMKz10/
NIeWiu5T6CUVAgMBAAGjgbIwga8wDwYDVR0TAQH/BAUwAwEB/zAOBgNVHQ8BAf8E
BAMCAQYwbQYIKwYBBQUHAQwEYTBfoV2gWzBZMFcwVRYJaW1hZ2UvZ2lmMCEwHzAH
BgUrDgMCGgQUj+XTGoasjY5rw8+AatRIGCx7GS4wJRYjaHR0cDovL2xvZ28udmVy
aXNpZ24uY29tL3ZzbG9nby5naWYwHQYDVR0OBBYEFH/TZafC3ey78DAJ80M5+gKv
MzEzMA0GCSqGSIb3DQEBBQUAA4IBAQCTJEowX2LP2BqYLz3q3JktvXf2pXkiOOzE
p6B4Eq1iDkVwZMXnl2YtmAl+X6/WzChl8gGqCBpH3vn5fJJaCGkgDdk+bW48DW7Y
5gaRQBi5+MHt39tBquCWIMnNZBU4gcmU7qKEKQsTb47bDN0lAtukixlE0kF6BWlK
WE9gyn6CagsCqiUXObXbf+eEZSqVir2G3l6BFoMtEMze/aiCKm0oHw0LxOXnGiYZ
4fQRbxC1lfznQgUy286dUV4otp6F01vvpX1FQHKOtw5rDgb7MzVIcbidJ4vEZV8N
hnacRHr2lVz2XTIIM6RUthg/aFzyQkqFOFSDX9HoLPKsEdao7WNq
-----END CERTIFICATE-----
_EOF

box:/# update-ca-certificates
{% endhighlight %}

Ok, so you want to know how do we know if the server is compatible with PayPal's update from VeriSign G2 to G5 certificate update? If the following command shows the VeriSign G5 certificate you should be ready to test against the PayPal sandbox.

Debian/Ubuntu based

{% highlight text %}
awk -v cmd='openssl x509 -noout -subject' '/BEGIN/{close(cmd)};{print | cmd}' < /etc/ssl/certs/ca-certificates.crt | grep G5
{% endhighlight %}

RHEL/CentOS based

{% highlight text %}
awk -v cmd='openssl x509 -noout -subject' '/BEGIN/{close(cmd)};{print | cmd}' < /etc/pki/tls/certs/ca-bundle.crt | grep G5
{% endhighlight %}
