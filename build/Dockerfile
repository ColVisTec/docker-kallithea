FROM ubuntu:20.04

ARG DEBIAN_FRONTEND=noninteractive

# Pass the path in advance to the directory where kallithea is installed.
ENV PATH $PATH:/home/kallithea/.local/bin

# set locale
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8

RUN : This is what to keep installed in the image. \
 && apt-get update \
 && apt-get install -y --no-install-recommends \
                    dumb-init ssh curl locales ca-certificates mercurial git python3 libpq5 libmysqlclient21 busybox-static \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

RUN : Prepare kallithea execution user. \
 && groupadd kallithea \
 && useradd -m -c '' -g kallithea kallithea

# Package version when installing by pip. ex.) 0.5.0
ARG KALLITHEA_VER=0.7.0

RUN : This is only needed for kallithea installation. \
 && curl -sL https://deb.nodesource.com/setup_14.x | bash - \
 && apt-get install -y --no-install-recommends \
                    build-essential libffi-dev \
                    python3-dev libpq-dev libmysqlclient-dev \
                    nodejs python3-pip \
 && : Build su-exec \
 && git -C /tmp clone https://github.com/ncopa/su-exec.git su-exec \
 && make -C /tmp/su-exec all \
 && mv /tmp/su-exec/su-exec /usr/bin/ \
 && rm -rf /tmp/su-exec \
 && : Install kallithea and optional packages.\
 && python3 -m pip install --no-cache-dir \
                           kallithea${KALLITHEA_VER:+==$KALLITHEA_VER} \
                           psycopg2 \
                           mysqlclient \
 && : Preparing the front-end files. \
 && kallithea-cli front-end-build \
 && : Clean up installation materials. \
 && apt-get purge -y \
                  build-essential libffi-dev \
                  python3-dev libpq-dev libmysqlclient-dev \
                  nodejs python3-pip \
 && apt-get autoremove -y \
 && apt-get clean \
 && rm -r  /etc/apt/sources.list.d/nodesource.list \
 && rm -rf /var/lib/apt/lists/* \
 && rm -rf /home/kallithea/.npm

# SSH setting
RUN sed -ri "s/^\\s*#?\\s*PubkeyAuthentication\\s+.+\$/PubkeyAuthentication yes/1" /etc/ssh/sshd_config \
 && sed -ri "s/^\\s*#?\\s*PasswordAuthentication\\s+.+\$/PasswordAuthentication no/1" /etc/ssh/sshd_config \
 && sed -ri "s/^\\s*#?\\s*ChallengeResponseAuthentication\\s+.+\$/ChallengeResponseAuthentication no/1" /etc/ssh/sshd_config \
 && sed -ri "s/^\\s*#?\\s*PermitRootLogin\\s+.+\$/PermitRootLogin no/1" /etc/ssh/sshd_config \
 && sed -ri "s/^\\s*#?\\s*X11Forwarding\\s+.+\$/X11Forwarding no/1" /etc/ssh/sshd_config

# Prepare a directory for storing persistent data.
RUN mkdir -p /kallithea/config \
 && mkdir -p /kallithea/repos \
 && mkdir -p /home/kallithea/.ssh \
 && chown kallithea:kallithea /kallithea/config \
 && chown kallithea:kallithea /kallithea/repos \
 && chown kallithea:kallithea /home/kallithea/.ssh

# Copy assets.
COPY ./assets/startup.sh /kallithea/startup.sh

# Service port
EXPOSE 5000 22

# Startup command
CMD ["dumb-init", "bash", "/kallithea/startup.sh"]
