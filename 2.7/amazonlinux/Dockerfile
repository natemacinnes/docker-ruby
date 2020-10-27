FROM amazonlinux:latest

RUN yum -y update

## BUILD RUBY
# skip installing gem documentation
RUN set -eux; \
  mkdir -p /usr/local/etc; \
  { \
    echo 'install: --no-document'; \
    echo 'update: --no-document'; \
  } >> /usr/local/etc/gemrc

ENV LANG C.UTF-8
ENV RUBY_MAJOR 2.7
ENV RUBY_VERSION 2.7.2
ENV RUBY_DOWNLOAD_SHA256 1b95ab193cc8f5b5e59d2686cb3d5dcf1ddf2a86cb6950e0b4bdaae5040ec0d6

# some of ruby's build scripts are written in ruby
#   we purge system ruby later to make sure our final image uses what we just built
# readline-dev vs libedit-dev: https://bugs.ruby-lang.org/issues/11869 and https://github.com/docker-library/ruby/issues/75
RUN set -eux; \
  \
  yum install -y \
  autoconf \
  bison \
  bzip2 \
  bzip2-devel \
  ca-certificates \
  coreutils \
  gcc \
  gdbm-devel \
  glib2-devel \
  glibc-devel \
  libffi-devel \
  libxml2-devel \
  libxslt-devel \
  kernel-headers \
  make \
  ncurses-devel \
  openssl \
  openssl-devel \
  patch \
  procps \
  readline-devel \
  ruby \
  tar \
  wget \
  xz \
  libyaml-devel \
  zlib-devel \
  ; \
  \
  wget -O ruby.tar.xz "https://cache.ruby-lang.org/pub/ruby/${RUBY_MAJOR%-rc}/ruby-$RUBY_VERSION.tar.xz"; \
  echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.xz" | sha256sum --check --strict; \
  \
  mkdir -p /usr/src/ruby; \
  tar -xJf ruby.tar.xz -C /usr/src/ruby --strip-components=1; \
  rm ruby.tar.xz; \
  \
  cd /usr/src/ruby; \
  # hack in "ENABLE_PATH_CHECK" disabling to suppress:
  #   warning: Insecure world writable dir
  { \
  echo '#define ENABLE_PATH_CHECK 0'; \
  echo; \
  cat file.c; \
  } > file.c.new; \
  mv file.c.new file.c; \
  \
  autoconf; \
  gnuArch="$(arch)"; \
  # the configure script does not detect isnan/isinf as macros
  export ac_cv_func_isnan=yes ac_cv_func_isinf=yes; \
  ./configure \
  --disable-install-doc \
  --enable-shared \
  ; \
  make; \
  make install; \
  cd /; \
  rm -r /usr/src/ruby; \
  # verify we have no "ruby" packages installed
  yum remove ruby -y; \
  [ "$(command -v ruby)" = '/usr/local/bin/ruby' ] echo "Yes" || echo "No" ;\
  export PATH=$PATH:/usr/local/bin ;\
  # rough smoke test
  ruby --version; \
  gem --version; \
  bundle --version

# don't create ".bundle" in all our apps
ENV GEM_HOME /usr/local/bundle
ENV BUNDLE_SILENCE_ROOT_WARNING=1 \
  BUNDLE_APP_CONFIG="$GEM_HOME"
ENV PATH $GEM_HOME/bin:$PATH
# adjust permissions of a few directories for running "gem install" as an arbitrary user
RUN mkdir -p "$GEM_HOME" && chmod 777 "$GEM_HOME"

CMD ["irb"]
