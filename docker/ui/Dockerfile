FROM opendatacube/datacube-core:1.7

ARG PORT="80"
ARG DB_HOSTNAME="127.0.0.1"
ARG DB_DATABASE="datacube"
ARG DB_USER="dc_user"
ARG DB_PASSWORD="localuser1234"
ARG DB_PORT="5432"
ARG ADMIN_EMAIL=""
ARG WORKDIR="/app"
ENV PORT=${PORT} \
    DB_HOSTNAME=${DB_HOSTNAME} \
    DB_DATABASE=${DB_DATABASE} \
    DB_USER=${DB_USER} \
    DB_PASSWORD=${DB_PASSWORD} \
    DB_PORT=${DB_PORT} \
    ADMIN_EMAIL=${ADMIN_EMAIL}

WORKDIR $WORKDIR
USER root
ENV DC_UI_DIR=${WORKDIR} \
    DC_UI_PYTHONHOME="/usr/bin/python3" \
	DC_UI_PYTHONPATH="/usr/lib/python3"

# Install system dependencies.
RUN apt-get update && apt-get install -y \
    apache2 \
    imagemagick \
    libapache2-mod-wsgi-py3 \
    libfreeimage3 \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies.
RUN pip3 install --upgrade pip && pip3 install \
    hdmedians lcmap-pyccd==2017.6.8 rasterio \
    numpy xarray scipy \
    sklearn scikit-image \
    shapely cloudpickle Cython netcdf4 \
    sqlalchemy psycopg2-binary \
    matplotlib seaborn \
    stringcase imageio \
    django==1.11.27 django-bootstrap3 \
    celery redis \
    && rm -rf /root/.cache

# Setup the Apache configuration.
COPY config/dc_ui.conf config/dc_ui.conf
#RUN sed -i "s/home\/localuser/root/g" config/dc_ui.conf && \
#    sed -i "s/Datacube\/data_cube_ui//g"
RUN cp config/dc_ui.conf /etc/apache2/sites-available/dc_ui.conf
RUN a2dissite 000-default.conf
RUN a2ensite dc_ui.conf
RUN service apache2 restart

# Create a directory to store files for the UI (notably task outputs).
RUN mkdir -p /datacube/ui_results && chmod 777 /datacube/ui_results

# Install postfix and mailutils.
# Credit to https://serverfault.com/a/144010/367947 and
#           https://serverfault.com/a/143982/367947.
#RUN debconf-set-selections << "postfix postfix/mailname string your.hostname.com" && \
#    debconf-set-selections << "postfix postfix/main_mailer_type string 'Internet Site'"
#RUN DEBIAN_FRONTEND=noninteractive apt-get install -y postfix mailutils

# Setup the Open Data Cube configuration.
COPY config/.datacube.conf config/.datacube.conf
RUN echo "\
[datacube] \n\
db_hostname: ${DB_HOSTNAME} \n\
db_database: ${DB_DATABASE} \n\
db_username: ${DB_USER} \n\
db_password: ${DB_PASSWORD} \n" > config/.datacube.conf
RUN cp config/.datacube.conf /etc/.datacube.conf

# Postgres pgpass configuration.
#RUN echo "${DB_HOSTNAME}:${DB_PORT}:${DB_DATABASE}:${DB_USER}:${DB_PASSWORD}" > config/.pgpass
#RUN cp config/.pgpass ~/.pgpass
#RUN chmod 600 ~/.pgpass

# Copy remaining files.
COPY manage.py manage.py
COPY db_backups/init_database.json db_backups/init_database.json
COPY data_cube_ui data_cube_ui
COPY apps apps
COPY utils utils

# Perform some runtime setup.
COPY docker/ui/config_setup.sh docker/ui/config_setup.sh
ENTRYPOINT ["docker/ui/config_setup.sh"]
# Sleep to keep the container from exiting so we can `exec` it.
CMD ["python3", "manage.py", "runserver", "0.0.0.0:${PORT}"]
