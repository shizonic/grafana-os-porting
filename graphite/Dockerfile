FROM graphiteapp/graphite-statsd
# TODO: Add tag

ARG paths='/opt /var/log /etc/collectd /etc/logrotate.d /etc/nginx /etc/redis /etc/run_once /etc/service /var/lib'

RUN chgrp -R 0 ${paths} && \
    chmod -R g=u ${paths}

# QUESTION: should i add user here?

