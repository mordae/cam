# Camera

## Workflow

We have several processes utilizing `systemd` that together work to get the
camera data in a sensible format to a permanent storage, provide snapshot of
the current feed data and react to motion detection.


### Feeder

Feeder is a simple vlc process connecting to the camera, forwarding the data
to a local worker process via UDP:

    cvlc rtsp://admin:airlive@10.0.250.139/media.amp?streamprofile=Profile2 \
        vlc://quit -I rc --rc-host=127.0.0.1:1234 \
        --snapshot-path=/tmp/snap.jpeg --snapshot-format=jpg \
        --sout '#duplicate{dst=standard{access=udp,mux=ts,dst=127.0.0.1:4321}}'


### Keeper

Keeper is a Python daemon that consumes data from Feeder and stores them in
a circular buffer (or something like that).  Whenever a motion event from
camera arrives on it's designated port, it starts a muxer and flushes the
buffer plus any additional data as they arrive to an output file.

The stream from the camera is getting saved until no motion events arrive
for some time.

Keeper also pokes it's source vlc daemon every few seconds to save a snapshot
of the stream for storage and preview.


### Web Server

Presents page with snapshots and directory listing with recordings, all with
SSL encryption and password-protected access.


### Cron

Periodic scripts decimate the snapshots and remove old recordings to ensure
enough space on the permanent storage.


