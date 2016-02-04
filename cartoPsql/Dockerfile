FROM ubuntu:12.04 

# Postgres
#RUN echo 'deb http://ppa.launchpad.net/cartodb/postgresql-9.3/ubuntu precise main' >> /etc/apt/sources.list
#RUN echo 'deb-src http://ppa.launchpad.net/cartodb/postgresql-9.3/ubuntu precise main' >> /etc/apt/sources.list

RUN apt-get update && apt-get install -y software-properties-common \
					 python-software-properties 
RUN add-apt-repository -y ppa:cartodb/postgresql-9.3
RUN apt-get update && apt-get install -y git \ 
					 libpq5 \
                     			 libpq-dev \
                     			 postgresql-client-9.3 \
                     			 postgresql-client-common \
					 postgresql-9.3 \
                     			 postgresql-contrib-9.3 \
                     			 postgresql-server-dev-9.3 \
                     			 postgresql-plpython-9.3 \
					 sudo
RUN add-apt-repository -y ppa:cartodb/pg-schema-trigger && apt-get update
RUN apt-get install -y postgresql-9.3-pg-schema-triggers

# GIS dependencies
RUN add-apt-repository -y ppa:cartodb/gis && apt-get update
RUN apt-get install -y  proj \ 
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

# Add users, being root isn't fun
RUN useradd -ms /bin/bash postgres
RUN echo "postgres   ALL=NOPASSWD: ALL" >> /etc/sudoers
#RUN sudo service postgresql restart

# Copy pg config file to proper dir
COPY postgresql-9.3_main/pg_hba.conf /etc/postgresql/9.3/main/pg_hba.conf
RUN service postgresql restart

# add postgres users
USER postgres
RUN bash -c "createuser publicuser --no-createrole --no-createdb --no-superuser -U postgres;"
RUN bash -c "createuser tileuser --no-createrole --no-createdb --no-superuser -U postgres;"
ENV PSQL_HOME /home/postgres
ENV PSQL_CDB $PSQL_HOME/cartodb-postgresql
RUN mkdir -p $PSQL_CDB
RUN bash -c "git clone https://github.com/CartoDB/cartodb-postgresql.git ${PSQL_CDB};"
RUN bash -c "cd ${PSQL_CDB} && sudo make all install"

# Initialize template postgis db
RUN sudo createdb -T template0 -O postgres -U postgres -E UTF8 template_postgis
RUN sudo createlang plpgsql -U postgres -d template_postgis
RUN psql -U postgres template_postgis -c 'CREATE EXTENSION postgis;CREATE EXTENSION postgis_topology;'
RUN sudo ldconfig
RUN sudo service postgresql restart


