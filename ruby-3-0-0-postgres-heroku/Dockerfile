# Dazzle do not supports ARG TRIGGER_REBUILD=2
FROM gitpod/workspace-base:latest

### Ruby ###
LABEL dazzle/layer=lang-ruby
LABEL dazzle/test=tests/lang-ruby.yaml
USER gitpod
RUN curl -fsSL https://rvm.io/mpapis.asc | gpg --import - \
    && curl -fsSL https://rvm.io/pkuczynski.asc | gpg --import - \
    && curl -fsSL https://get.rvm.io | bash -s stable \
    && bash -lc " \
        rvm requirements \
        && rvm install 3.0.0 \
        && rvm use 3.0.0 --default \
        && rvm rubygems current \
        && gem install bundler --no-document \
        && gem install solargraph --no-document" \
    && echo '[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*' >> /home/gitpod/.bashrc.d/70-ruby
RUN echo "rvm_gems_path=/workspace/.rvm" > ~/.rvmrc

USER gitpod

# Docker build does not rebuild an image when a base image is changed, increase this counter to trigger it.
ENV TRIGGER_REBUILD=1
# Install PostgreSQL
RUN sudo install-packages postgresql-12 postgresql-contrib-12

# Setup PostgreSQL server for user gitpod
ENV PATH="$PATH:/usr/lib/postgresql/12/bin"
ENV PGDATA="/workspace/.pgsql/data"
RUN mkdir -p ~/.pg_ctl/bin ~/.pg_ctl/sockets \
 && printf '#!/bin/bash\n[ ! -d $PGDATA ] && mkdir -p $PGDATA && initdb -D $PGDATA\npg_ctl -D $PGDATA -l ~/.pg_ctl/log -o "-k ~/.pg_ctl/sockets" start\n' > ~/.pg_ctl/bin/pg_start \
 && printf '#!/bin/bash\npg_ctl -D $PGDATA -l ~/.pg_ctl/log -o "-k ~/.pg_ctl/sockets" stop\n' > ~/.pg_ctl/bin/pg_stop \
 && chmod +x ~/.pg_ctl/bin/*
ENV PATH="$PATH:$HOME/.pg_ctl/bin"
ENV DATABASE_URL="postgresql://gitpod@localhost"
ENV PGHOSTADDR="127.0.0.1"
ENV PGDATABASE="postgres"

# This is a bit of a hack. At the moment we have no means of starting background
# tasks from a Dockerfile. This workaround checks, on each bashrc eval, if the
# PostgreSQL server is running, and if not starts it.
RUN printf "\n# Auto-start PostgreSQL server.\n[[ \$(pg_ctl status | grep PID) ]] || pg_start > /dev/null\n" >> ~/.bashrc

# Heroku
RUN curl https://cli-assets.heroku.com/install-ubuntu.sh | sh