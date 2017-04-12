# Nominatim

* [Docker hub repository](https://hub.docker.com/r/hbaltz/nominatim/)


## Description

Nominatim : nomatim preinstalled without data, see add data to learn to it.

## Usage

### Run the image

! This image doesn't have entrypoint

* Pull the image `docker pull hbaltz/nominatim`
* Run a python command `docker run hbaltz/nominatim bash` to go in the cointainer

### Compile the image yourself

* Clone the git repository [https://github.com/hbaltz/docker-nominatim.git]( https://github.com/hbaltz/docker-nominatim.git)
* In the repository, launch the compilation : `docker build -t <the-tag-you-want> ./`

## Add data

 * Create a Dockerfile `sudo nano Dockerfile`
 * Go to [Geofrabik](http://download.geofabrik.de/), choose the data you want to downlaod (copy the link to download it)
 * Fill the Dockerfile :
 
```
FROM hbaltz/nominatim
MAINTAINER Hugo BALTZ <hugobaltz@gmail.com>

# Load initial data
ENV PBF_DATA http://download.geofabrik.de/africa/angola-latest.osm.pbf # The data you want on geofrabik in pbf format

RUN curl $PBF_DATA --create-dirs -o /app/src/data.osm.pbf

RUN service postgresql start && \
    sudo -u postgres psql postgres -tAc "SELECT 1 FROM pg_roles WHERE rolname='nominatim'" | grep -q 1 || sudo -u postgres createuser -s nominatim && \
    sudo -u postgres psql postgres -tAc "SELECT 1 FROM pg_roles WHERE rolname='www-data'" | grep -q 1 || sudo -u postgres createuser -SDR www-data && \
    sudo -u postgres psql postgres -c "DROP DATABASE IF EXISTS nominatim" && \
    useradd -m -p password1234 nominatim && \
    chown -R nominatim:nominatim ./src && \
    sudo -u nominatim ./src/utils/setup.php --osm-file /app/src/data.osm.pbf --all --threads 2 && \
    service postgresql stop

COPY start.sh /app/start.sh # Recover it from this repositry
CMD /app/start.sh
 ```

## Summary

A docker image with nominatim installed

