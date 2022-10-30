## 1. nginx
```
FROM lisywork/lisyubuntu20
RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/*
RUN cd / && \
    echo '#!/bin/bash' > start.sh && \
    echo '/usr/sbin/nginx -g "daemon off;"' > start.sh
WORKDIR /
CMD ["sh", "start.sh"]


```
