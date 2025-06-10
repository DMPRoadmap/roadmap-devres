# Use official Ruby base with version 3.0.4
FROM ruby:3.0.4

# Set environment variables
ENV RAILS_ENV=development \
    BUNDLER_VERSION=2.4.22 \
    LANG=C.UTF-8 \
    DEBIAN_FRONTEND=noninteractive

# Install Linux dependencies
RUN apt-get update -qq && apt-get install -y \
    build-essential \
    libpq-dev \
    postgresql-client \
    autoconf \
    automake \
    gawk \
    g++ \
    git \
    imagemagick \
    libffi-dev \
    libgdbm-dev \
    libreadline-dev \
    libssl-dev \
    libtool \
    libyaml-dev \
    curl \
    gnupg \
    shared-mime-info \
    vim \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js (v20 LTS) and Yarn
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get install -y nodejs && \
    npm install --global yarn

# Set working directory
WORKDIR /app

# Copy Gemfiles and install Ruby dependencies
COPY Gemfile Gemfile.lock ./
RUN gem install bundler -v "$BUNDLER_VERSION" && \
    bundle _${BUNDLER_VERSION}_ install

# Copy application source
COPY . .

# Install JS packages
RUN yarn install

# Precompile assets (optional in dev)
# RUN bin/rails assets:precompile

# Expose default Rails port
EXPOSE 3000

# Start development server
CMD ["bin/dev"]

