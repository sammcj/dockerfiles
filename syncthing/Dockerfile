FROM ubuntu:trusty
# Dockerfile to run Syncthing. Requires an external container manager such as systemd to auto-restart the container when Syncthing restarts.
MAINTAINER Scott Hansen <firecat4153@gmail.com>

RUN apt-get update && apt-get install -qy supervisor ca-certificates
RUN useradd --no-create-home -g users --uid 22000 syncthing
ADD https://github.com/syncthing/syncthing/releases/download/v0.10.29/syncthing-linux-amd64-v0.10.29.tar.gz /syncthing.tar.gz
RUN tar -xzvf syncthing.tar.gz && mv sync*/syncthing /syncthing && rm syncthing.tar.gz
# To use an existing syncthing binary in the root directory, comment out the previous 2 lines and uncomment the next line.
# ADD syncthing /syncthing

USER syncthing

CMD ["/syncthing", "-home=/config"]
