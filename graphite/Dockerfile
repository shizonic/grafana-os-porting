ARG tag=latest
FROM graphiteapp/graphite-statsd:$tag

ARG paths='/opt /var/log /etc/collectd /etc/logrotate.d /etc/nginx /etc/redis /etc/run_once /etc/service /var/lib /usr/sbin /run/nginx.pid'
RUN touch /run/nginx.pid && \
    chgrp -R 0 ${paths} && \
    chmod -R g=u ${paths}

ARG user=nginx
RUN adduser $user root

USER $user
