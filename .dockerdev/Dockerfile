ARG RUBY_VERSION
FROM ruby:$RUBY_VERSION

ARG PG_MAJOR
ARG NODE_MAJOR
ARG BUNDLER_VERSION
ARG YARN_VERSION

# ソースリストにPostgreSQLを追加
RUN curl -sSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -\
  && echo 'deb http://apt.postgresql.org/pub/repos/apt/ stretch-pgdg main' $PG_MAJOR > /etc/apt/sources.list.d/pgdg.list

# ソースリストにNodeJSを追加
RUN curl -sL https://deb.nodesource.com/setup_$NODE_MAJOR.x | bash -

# ソースリストにYarnを追加
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -\
  && echo 'deb http://dl.yarnpkg.com/debian/ stable main' > /etc/apt/sources.list.d/yarn.list

# 依存関係をインストール
COPY .dockerdev/Aptfile /tmp/Aptfile
RUN apt-get update -qq && DEBIAN_FRONTEND=noninteractive apt-get -yq dist-upgrade &&\
  DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends\
    build-essential\
    nodejs\
    yarn=$YARN_VERSION-1\
    $(cat /tmp/Aptfile | xargs) &&\
    apt-get clean &&\
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* &&\
    truncate -s 0 /var/log/*log

# Chromeをインストール
# ref. https://github.com/CircleCI-Public/circleci-dockerfiles/blob/ea84700d9201c57e3641638dd725e931d1f18bb9/ruby/images/2.6.5-stretch/node-browsers/Dockerfile
RUN curl --silent --show-error --location --fail --retry 3 --output /tmp/google-chrome-stable_current_amd64.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb \
    && apt-get update -qq \
    && (dpkg -i /tmp/google-chrome-stable_current_amd64.deb || apt-get -fy install)  \
    && sed -i 's|HERE/chrome"|HERE/chrome" --disable-setuid-sandbox --no-sandbox|g' \
        "/opt/google/chrome/google-chrome" \
    && google-chrome --version \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* \
    && truncate -s 0 /var/log/*log

# chrome driverをインストール
RUN CHROME_VERSION="$(google-chrome --version)" \
    && export CHROMEDRIVER_RELEASE="$(echo $CHROME_VERSION | sed 's/^Google Chrome //')" && export CHROMEDRIVER_RELEASE=${CHROMEDRIVER_RELEASE%%.*} \
    && CHROMEDRIVER_VERSION=$(curl --silent --show-error --location --fail --retry 4 --retry-delay 5 http://chromedriver.storage.googleapis.com/LATEST_RELEASE_${CHROMEDRIVER_RELEASE}) \
    && curl --silent --show-error --location --fail --retry 4 --retry-delay 5 --output /tmp/chromedriver_linux64.zip "http://chromedriver.storage.googleapis.com/$CHROMEDRIVER_VERSION/chromedriver_linux64.zip" \
    && cd /tmp \
    && unzip chromedriver_linux64.zip \
    && rm -rf chromedriver_linux64.zip \
    && mv chromedriver /usr/local/bin/chromedriver \
    && chmod +x /usr/local/bin/chromedriver \
    && chromedriver --version

# bundlerとPATHを設定
ENV LANG=C.UTF-8\
  GEM_HOME=/bundle\
  BUNDLE_JOBS=4\
  BUNDLE_RETRY=3
ENV BUNDLE_PATH $GEM_HOME
ENV BUNDLE_APP_CONFIG=$BUNDLE_PATH\
  BUNDLE_BIN=$BUNDLE_PATH/bin
ENV PATH /app/bin:$BUNDLE_BIN:$PATH

# RubyGemsをアップグレードして必要なバージョンのbundlerをインストール
RUN gem update --system &&\
    gem install bundler:$BUNDLER_VERSION

WORKDIR /app