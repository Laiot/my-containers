FROM fedora:latest
LABEL maintainer "Laiot <carmelo.sarta.main@gmail.com>"

# Install Node 
## I really wanted to use nvm as I usually do but I'm having problem with using npm later on.
## TODO edit and build from source
RUN dnf install -y nodejs

# Install Ruby 
RUN dnf install -y ruby

# Install and Setup Node and Ruby necessary libraries
RUN npm install -g npm@latest && \
	npm install -g yarn && \
	gem install bundler  
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