---
title: How To - Setup nginx as a caching reverse proxy
---

I recently had a project where 97% of a customers server was serving up static content. CDN was not a option because the customers was concerned about caching frequently updated images. The existing problem was Apache was bloated with modules and even these lightweight requests were causing the server to flap. The compromise was to put nginx in-front of Apache and let it directly serve the static content directly.

First install nginx,
