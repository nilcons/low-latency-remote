FROM debian:buster AS spice-gtk
RUN /bin/echo -e 'deb-src http://deb.debian.org/debian buster main\ndeb-src http://security.debian.org/debian-security buster/updates main\ndeb-src http://deb.debian.org/debian buster-updates main' >/etc/apt/sources.list.d/sources.list
RUN apt-get update && \
    apt-get install -y build-essential && \
    apt-get build-dep -y spice-gtk
WORKDIR /app
RUN apt-get source spice-gtk
COPY 0003-spice-gtk-more-buttons.patch /app/spice-gtk-0.35/debian/patches
WORKDIR /app/spice-gtk-0.35
RUN echo 0003-spice-gtk-more-buttons.patch >>debian/patches/series
RUN dpkg-buildpackage -us -uc

FROM debian:buster
RUN apt-get update
WORKDIR /app
RUN apt-get install --no-install-recommends -y virt-viewer && apt-get clean
COPY --from=spice-gtk /app/libspice-client-gtk-3.0-5_*deb ./
RUN dpkg -i libspice-client-gtk-3.0-5_*deb
# The sleep is necessary, because Docker has a race condition:
# if the container exits too fast, then you get:
# "failed to resize tty, using default size"
ENTRYPOINT ["bash", "-c", "\
  if [ -z \"$1\" ]; then\
    echo \"Usage: docker run -it --rm --network=host --ipc=host --env=DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix:rw -v ~/.Xauthority:/root/.Xauthority:ro nilcons/spice-client spice://1.2.3.4:5900/\";\
  else \
    remote-viewer \"$@\"; \
  fi; sleep 1;", "we are all bash"]
