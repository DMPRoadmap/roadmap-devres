FROM ruby:3.3.1

# Dependancies
RUN apt-get update -qq && \
  apt-get install -y \
  build-essential \
  git \
  libgmp3-dev \
  libpq-dev \
  postgresql-client \
  gettext \
  certbot \
  vim \
  ack

ARG INSTALL_PATH=/usr/src/app
ENV INSTALL_PATH $INSTALL_PATH
ENV BUNDLE_PATH=/bundle/ \
    BUNDLE_BIN=/bundle/bin \
    GEM_HOME=/bundle
ENV Path="${BUNDLE_BIN}:${PATH}"

WORKDIR $INSTALL_PATH

RUN touch ~/.bashrc
RUN echo "alias rails='bundle exec rails'" >> ~/.bashrc
RUN echo "export EDITOR=vim" >> ~/.bashrc
RUN echo "alias racp='rails assets:clobber assets:precompile'" >> ~/.bashrc

# install yarn+node from packages
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN curl -sL https://deb.nodesource.com/setup_18.x | bash
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get install -y nodejs
RUN apt-get update && apt-get install -y yarn
RUN wget --quiet https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.3/wkhtmltox-0.12.3_linux-generic-amd64.tar.xz && \
    tar vxf wkhtmltox-0.12.3_linux-generic-amd64.tar.xz && \
    cp wkhtmltox/bin/wk* /usr/local/bin/ && \
    rm -rf wkhtmltox


# re-build from here if Gemfile or .lock change
COPY ./roadmap/Gemfile ./
RUN gem install bundler
RUN bundle config set without 'mysql thin'
RUN bundle install
RUN yarn install

# expose correct port
EXPOSE 3000

# Start the server
CMD ["bundle", "exec", "puma", "-C", "config/puma.rb"]


