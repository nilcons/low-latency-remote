FROM debian:buster AS libspice
RUN /bin/echo -e 'deb-src http://deb.debian.org/debian buster main\ndeb-src http://security.debian.org/debian-security buster/updates main\ndeb-src http://deb.debian.org/debian buster-updates main' >/etc/apt/sources.list.d/sources.list
RUN apt-get update && \
    apt-get install -y git build-essential && \
    apt-get build-dep -y spice
WORKDIR /app
RUN apt-get source spice
COPY libspice-more-buttons.patch /app/spice-0.14.0/debian/patches
WORKDIR /app/spice-0.14.0
RUN echo libspice-more-buttons.patch >>debian/patches/series
RUN dpkg-buildpackage -us -uc

FROM debian:buster AS x11spice
RUN /bin/echo -e 'deb-src http://deb.debian.org/debian buster main\ndeb-src http://security.debian.org/debian-security buster/updates main\ndeb-src http://deb.debian.org/debian buster-updates main' >/etc/apt/sources.list.d/sources.list
RUN apt-get update && \
    apt-get install -y git build-essential && \
    apt-get build-dep -y spice
WORKDIR /app
COPY --from=libspice /app/*deb ./
RUN apt-get install -y ./*deb  xutils-dev libxcb-damage0-dev libxcb-xtest0-dev libxcb-shm0-dev libxcb-util0-dev libxcb-xkb-dev libgtk-3-dev
COPY x11spice-more-buttons.patch /app
RUN git clone https://gitlab.freedesktop.org/spice/x11spice.git
WORKDIR /app/x11spice
RUN patch -p1 <../x11spice-more-buttons.patch && ./autogen.sh --prefix=/app/x11spice-bin && make install

FROM debian:buster
RUN apt-get update
WORKDIR /app
COPY --from=libspice /app/libspice-server1_*deb ./
COPY --from=x11spice /app/x11spice-bin/bin/x11spice ./
RUN apt-get install --no-install-recommends -y ./*deb libxcb-damage0 libxcb-xtest0 libxcb-shm0 libxcb-util0 libxcb-xkb1 libxcb-xfixes0 libgtk-3-0 && apt-get clean
# The sleep is necessary, because Docker has a race condition:
# if the container exits too fast, then you get:
# "failed to resize tty, using default size"
ENTRYPOINT ["bash", "-c", "\
  if [ -z \"$1\" ]; then\
    echo \"Usage: docker run -it --rm --network=host --ipc=host -v /tmp/.X11-unix:/tmp/.X11-unix:rw -v ~/.Xauthority:/root/.Xauthority:ro nilcons/spice-server --password=topsecret\";\
  else \
    ./x11spice --display=:0 --allow-control --hide \"$@\"; \
  fi; sleep 1;", "we are all bash"]
