# Low latency remote

Our goal here is to have a remote desktop solution.

Requirements:
  - sharing of the existing desktop session
  - NOT a new virtual, terminal server session
  - the two machines that are close to each other (<5 ms)
  - both running modern Debian GNU/Linux
  - support multi-monitor support
  - support audio
  - support USB forwarding (Yubikey)
  - support video playback at a reasonable quality (Youtube)
  - somewhat secure

Networking is handled with IPsec already in my environment, so I will
not need printer/scanner/etc forwarding, as I already have IPv4
addressing working correctly between the two locations.

Previous solution was the free edition of Nomachine, but they have at
least 30-40ms of lag on my 1ms link network link, so let's try to do
better and this time use free software only!

## Basic building block: video+mouse/keyboard

We will use [SPICE](https://www.spice-space.org/) for our basic remote
desktop tool.  This is a newer protocol than VNC, so a little bit less
tested, but in exchange it has a heuristic that switches between some
traditional 2D encoding and VP8 when Video is being watched.

This heuristics makes it pretty nice in practice, for 2D applications
we have very good latency, and we have proper performance also when
watching video.

The keyboard support is also quite good in
[x11spice](https://gitlab.freedesktop.org/spice/x11spice) by default:
the keys are just seem to be sent over as keycodes, without taking
into account the local keymap or anything.  `xev` shows that even
alone modifier presses/releases are correctly forwarded.  Key repeat
works out of the box.

### Mouse trouble with BACK and FOWARD buttons

There is only one problem with SPICE, and that is the mouse support:
only 3 mouse buttons and the vertical scroll is supported.  I can live
without horizontal scroll (actually I don't have that on my mouse),
but BACK and FORWARD I really do need.  Under Linux/X11 these are
mouse button 8 and 9, which is not supported either by VNC nor by
SPICE out of the box.

Looking into the code, it looked like a big mess and coming up with a
proper contribution would require a lot of engineering time of my own
and usually it's very hard to get patches into open source software if
the maintainers are even a little bit disagreeable.  I'm not saying
the SPICE guys are, but I have been burn before, so I will just keep
this to myself this time.

You can find the server build process in `spice/server/Dockerfile` and
the client in `spice/client/Dockerfile`.

The server can be built and ran like this:

    cd spice/server
    docker build -t nilcons/spice-server .
    docker run -it --rm --network=host --ipc=host \
      -v /tmp/.X11-unix:/tmp/.X11-unix:rw -v ~/.Xauthority:/root/.Xauthority:ro \
      nilcons/spice-server --password=topsecret

Network and IPC access is necessary for communicating with the X
server.  Remember: we are using Docker here not for security, but so
that we avoid all the drama of contributing back to open source.

But make sure, you are running this securely from the network
standpoint: this will expose port 5900/tcp, that has to be tunneled
with SSH or some other tech (IPsec, VPN, etc.).

On the client side:

    cd spice/client
    docker build -t nilcons/spice-client .
    docker run -it --rm --network=host --ipc=host --env=DISPLAY \
      -v /tmp/.X11-unix:/tmp/.X11-unix:rw -v ~/.Xauthority:/root/.Xauthority:ro \
      nilcons/spice-client spice://1.2.3.4:5900/
