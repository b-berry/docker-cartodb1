FROM ubuntu:12.04

# Set env var
RUN DEBIAN_FRONTEND=noninteractive

# Install core packages
RUN apt-get update &&\ 
    apt-get install -y software-properties-common \
        python-software-properties

# Update APT sources
RUN add-apt-repository -y ppa:cartodb/redis &&\ 
    add-apt-repository -y ppa:cartodb/nodejs-010 &&\
    apt-get update

# Add packages
RUN apt-get update &&\ 
    apt-get install -y autoconf \
        binutils-doc \
        bison \
        build-essential \
        flex
        git \
        imagemagick \
        libicu-dev \
        libpq5 \
        libpq-dev \
        libreadline6-dev \ 
        make \
        nodejs \
        openssl \
        python-all-dev \
        redis-server \
        sudo \
        wget \ 
        unp \
        zip

ENV PROJ_SRC /src
ENV PROJ_SRV /srv

RUN mkdir -p $PROJ_SRV
WORKDIR $PROJ_SRV

# Install && configure Ruby
RUN wget -O ruby-install-0.5.0.tar.gz https://github.com/postmodern/ruby-install/archive/v0.5.0.tar.gz &&\
    tar -xzvf ruby-install-0.5.0.tar.gz &&\
    cd ruby-install-0.5.0/ &&\
    make install &&\
    sudo ruby-install ruby 2.2.3 &&\
    echo 'export PATH=$PATH:/opt/rubies/ruby-2.2.3/bin' >> ~/.bashrc
     
ENV RUBY_DIR /opt/rubies/ruby-2.2.3/bin

# Install Rubygems
RUN mkdir -p $PROJ_SRC && cd $PROJ_SRC &&\ 
    wget -O rubygems-2.5.2.tgz https://rubygems.org/rubygems/rubygems-2.5.2.tgz &&\
    tar -xzvf rubygems-2.5.2.tgz &&\
    cd rubygems-2.5.2 &&\
    $RUBY_DIR/ruby setup.rb 

# Install gems
RUN cd $PROJ_SRV/ruby-install-0.5.0/ &&\
    $RUBY_DIR/gem install bundler &&\
    $RUBY_DIR/gem install compass &&\
    cd ../

# Install Editor
RUN git clone --recursive https://github.com/CartoDB/cartodb.git &&\
    cd cartodb &&\
    sudo wget -O /tmp/get-pip.py https://bootstrap.pypa.io/get-pip.py &&\
    sudo python /tmp/get-pip.py &&\
    $RUBY_DIR/gem install pg -v '0.15.0' &&\
    $RUBY_DIR/gem install charlock_holmes -v '0.7.2' &&\ 
    RAILS_ENV=development $RUBY_DIR/bundle install

RUN cd cartodb &&\
    #npm config set registry http://registry.npmjs.org/ &&\
    npm install &&\
    sudo pip install --no-use-wheel -r python_requirements.txt &&\ 
    export PATH=$PATH:$PWD/node_modules/grunt-cli/bini &&\
    $RUBY_DIR/bundle install &&\
    $RUBY_DIR/bundle exec grunt --environment development &&\
    cp config/app_config.yml.sample config/app_config.yml &&\
    cp config/database.yml.sample config/database.yml 

RUN redis-server & &&\
    RAILS_ENV=development $RUBY_DIR/bundle exec rails server

ENTRYPOINT RAILS_ENV=development $RUBY_DIR/bundle exec ./script/resque





