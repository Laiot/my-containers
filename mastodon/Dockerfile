FROM fedora:latest
LABEL maintainer "Laiot <carmelo.sarta.main@gmail.com>"

# Installing general dependencies
RUN dnf install -y wget dpkg jemalloc-devel openssl && \
    dnf group install -y "Development Tools"

# Install Node LTS
ENV NODE_VER="18.12.1"
RUN ARCH= && \
    dpkgArch="$(dpkg --print-architecture)" && \
    case "${dpkgArch##*-}" in \
    amd64) ARCH='x64';; \
    ppc64el) ARCH='ppc64le';; \
    s390x) ARCH='s390x';; \
    arm64) ARCH='arm64';; \
    armhf) ARCH='armv7l';; \
    i386) ARCH='x86';; \
    *) echo "unsupported architecture"; exit 1 ;; \
    esac && \
    echo "Etc/UTC" > /etc/localtime && \
    wget -q https://nodejs.org/download/release/v$NODE_VER/node-v$NODE_VER-linux-$ARCH.tar.gz && \
	tar xf node-v$NODE_VER-linux-$ARCH.tar.gz && \
	rm node-v$NODE_VER-linux-$ARCH.tar.gz && \
	mv node-v$NODE_VER-linux-$ARCH /opt/node

# Install Ruby 
ENV RUBY_VER="3.0.4"
RUN wget https://cache.ruby-lang.org/pub/ruby/${RUBY_VER%.*}/ruby-$RUBY_VER.tar.gz && \
	tar xf ruby-$RUBY_VER.tar.gz && \
	cd ruby-$RUBY_VER && \
	./configure --prefix=/opt/ruby \
	  --with-jemalloc \
	  --with-shared \
	  --disable-install-doc && \
	make -j"$(nproc)" > /dev/null && \
	make install && \
	rm -rf ../ruby-$RUBY_VER.tar.gz ../ruby-$RUBY_VER

ENV PATH="${PATH}:/opt/ruby/bin:/opt/node/bin"

# Install and Setup Node and Ruby necessary libraries
RUN npm install -g npm@latest && \
	npm install -g yarn && \
	gem install bundler && \
    mkdir /opt/mastodon 
COPY Gemfile* package.json yarn.lock /opt/mastodon/
RUN cd /opt/mastodon && \
    bundle config set --local deployment 'true' && \
    bundle config set --local without 'development test' && \
    bundle config set silence_root_warning true && \
	bundle install -j"$(nproc)" && \
	yarn install --pure-lockfile

# Copy over all the langs needed for runtime
COPY --from=build-dep /opt/node /opt/node
COPY --from=build-dep /opt/ruby /opt/ruby
ENV PATH="${PATH}:/opt/mastodon/bin"

# Create the mastodon user
ARG UID=991
ARG GID=991
RUN echo "Etc/UTC" > /etc/localtime && \
	addgroup --gid $GID mastodon && \
	useradd -m -u $UID -g $GID -d /opt/mastodon mastodon && \
	echo "mastodon:$(head /dev/urandom | tr -dc A-Za-z0-9 | head -c 24 | mkpasswd -s -m sha-256)" | chpasswd

# Install mastodon runtime deps
#RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections
RUN ln -s /opt/mastodon /mastodon && \
	gem install bundler 

# Copy over mastodon source, and dependencies from building, and set permissions
COPY --chown=mastodon:mastodon . /opt/mastodon
COPY --from=build-dep --chown=mastodon:mastodon /opt/mastodon /opt/mastodon

# Run mastodon services in prod mode
ENV RAILS_ENV="production"
ENV NODE_ENV="production"

# Tell rails to serve static files
ENV RAILS_SERVE_STATIC_FILES="true"
ENV BIND="0.0.0.0"

# Set the run user
USER mastodon

# Precompile assets
RUN cd ~ && \
	OTP_SECRET=precompile_placeholder SECRET_KEY_BASE=precompile_placeholder rails assets:precompile && \
	yarn cache clean

# Set the work dir and the container entry point
WORKDIR /opt/mastodon
ENTRYPOINT ["/usr/bin/tini", "--"]
EXPOSE 3000 4000