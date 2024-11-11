FROM debian:buster-slim

# References: 
# https://wiki.debian.org/DebianRepository/SetupWithReprepro
# https://blog.packagecloud.io/how-to-create-debian-repository-with-reprepro/
# https://unix.stackexchange.com/questions/87130/how-to-quickly-create-a-local-apt-repository-for-random-packages-using-a-debian

WORKDIR /root

RUN apt update && apt install -y reprepro lighttpd openssh-server nano
RUN apt install -y procps

# Configure lighttpd
RUN cd /var/www/html/ && mkdir conf dists db incoming pool && rm index*
RUN echo 'server.dir-listing = "enable"' >> /etc/lighttpd/lighttpd.conf
RUN echo 'dir-listing.exclude = ( "conf", "db", "incoming", "index*" )' >> /etc/lighttpd/lighttpd.conf

# Configure repository
ENV REPREPRO_BASE_DIR=/var/www/html
RUN mkdir /deb_packages
COPY <<"EOF" /var/www/html/conf/distributions
Origin: Tv-box project
Label: Tv-box project
Codename: bookworm
Architectures: source amd64 arm64
Components: main
Description: Repositorio de teste para projeto de tv box
EOF

COPY <<"EOF" /var/www/html/conf/options
verbose
basedir /var/www/html
EOF

# The Repository
EXPOSE 8080
# SSH
EXPOSE 2022

CMD service ssh start && service lighttpd start && /bin/bash

