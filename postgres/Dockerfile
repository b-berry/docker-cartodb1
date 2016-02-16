FROM ubuntu:12.04 

# Set env var
RUN DEBIAN_FRONTEND=noninteractive

# Install core packages
RUN apt-get update &&\ 
    apt-get install -y software-properties-common \
        python-software-properties \
        make \
        sudo 

# Update APT sources
RUN add-apt-repository -y ppa:cartodb/gis &&\ 
    add-apt-repository -y ppa:cartodb/nodejs-010 &&\
    add-apt-repository -y ppa:cartodb/postgresql-9.3 &&\
    add-apt-repository -y ppa:cartodb/pg-schema-trigger &&\
    apt-get update

# Add cartodb packages 
RUN apt-get install -y autoconf \
        binutils-doc \ 
        bison \ 
        build-essential \
        flex \
        git \ 
        libpq5 \
        libpq-dev \
        nodejs \
        postgresql-client-9.3 \
        postgresql-client-common \
        postgresql-9.3 \
        postgresql-contrib-9.3 \
        postgresql-server-dev-9.3 \
        postgresql-plpython-9.3 \
        vim \
        zip

# Add postgres packages
RUN apt-get install -y postgresql-9.3-pg-schema-triggers \
        proj \ 
        proj-bin \
        proj-data \ 
        libproj-dev \
        libjson0 \
        libjson0-dev \
        python-simplejson \
        libgeos-c1v5 \ 
        libgeos-dev \
        libgeos-c1v5 \ 
        libgeos-dev \
        ogr2ogr2-static-bin \
        libxml2-dev \
        liblwgeom-2.1.8 \
        postgis \ 
        postgresql-9.3-postgis-2.2 \
        postgresql-9.3-postgis-scripts

# Add users if necessary, being root isn't fun
RUN id -u postgres || useradd -ms /bin/bash postgres
RUN echo "postgres   ALL=NOPASSWD: ALL" >> /etc/sudoers

# Copy pg config file to proper dir
COPY postgresql-9.3_main/pg_hba.conf /etc/postgresql/9.3/main/pg_hba.conf
COPY postgresql-9.3_main/postgresql.conf /etc/postgresql/9.3/main/postgresql.conf
#RUN service postgresql start

# Setup postgres log
#ENV PSQL_LOG /var/log/postgres
#RUN mkdir -p $PSQL_LOG &&\
#    chown -R postgres:postgres $PSQL_LOG

USER postgres
RUN  /etc/init.d/postgresql start &&\
    createuser publicuser --no-createrole --no-createdb --no-superuser -U postgres &&\
    createuser tileuser --no-createrole --no-createdb --no-superuser -U postgres

# Expose the PostgreSQL port
EXPOSE 5432

# Add VOLUMEs to allow backup of config, logs and databases
VOLUME  ["/etc/postgresql", "/var/log/postgresql", "/var/lib/postgresql"]

# Set up environment variables
ENV PSQL_HOME /home/postgres
ENV PSQL_CDB $PSQL_HOME/cartodb-postgresql
ENV PSQL_SRC $PSQL_HOME/src
ENV SQL_API $PSQL_SRC/CartoDB-SQL-API

# Checkout && build cartodb-postgresql
RUN sudo mkdir -p $PSQL_CDB && sudo chown -R postgres:postgres $PSQL_CDB &&\
    git clone https://github.com/CartoDB/cartodb-postgresql.git ${PSQL_CDB} &&\
    cd ${PSQL_CDB} && sudo make all install
   
 # Checkout && build cartodb-sql-api
RUN sudo mkdir -p $PSQL_SRC && sudo chown -R postgres:postgres $PSQL_SRC &&\ 
    cd ${PSQL_SRC} &&\
    git clone git://github.com/CartoDB/CartoDB-SQL-API.git &&\
    cd ${SQL_API} &&\
    git checkout master &&\
    npm install --quiet &&\
    cp config/environments/development.js.example config/environments/development.js &&\
    node app.js development &&\
    cd ${PSQL_HOME}

## Initialize template postgis db
# postgres -D /usr/local/pgsql/data >$PSQL_LOG/psql.log 2>&1 & \
RUN sudo service postgresql start &&\ 
    sudo createdb -T template0 -O postgres -U postgres -E UTF8 template_postgis &&\
    #sudo createlang plpgsql -U postgres -d template_postgis &&\
    psql -U postgres template_postgis -c 'CREATE EXTENSION postgis;CREATE EXTENSION postgis_topology;' &&\
    sudo ldconfig &&\
    sudo service postgresql restart

# Set the default command to run when starting the container
CMD ["/usr/lib/postgresql/9.3/bin/postgres", "-D", "/var/lib/postgresql/9.3/main", "-c", "config_file=/etc/postgresql/9.3/main/postgresql.conf"]
