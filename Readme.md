Collectd Docker base image
==========================
This is a Docker base image for [collectd](http://collectd.org) system statistics collection daemon.

The image includes several configuration snippets ready to use in your custom collectd images and it is intended to be used as a base image, it does nothing on its own. You can base your own collectd image on this one and pick your desired configuration snippets.

The available snippets are located at ```/etc/collectd/conf-available```. There are snippets for read and write plugin configurations and global options.

Usage example
-------------
Put this in your own Dockerfile:
```
FROM pataquets/collectd

# Build our custom collectd.conf file:
# Disable FQDN lookup to avoid collectd complaining.
# Read CPU, system load and vmem stats and print them to stdout.
RUN \
  cat \
    /etc/collectd/conf-available/global-FQDNLookup-disable.conf \
    /etc/collectd/conf-available/read-cpu.conf \
    /etc/collectd/conf-available/read-load.conf \
    /etc/collectd/conf-available/read-vmem.conf \
    /etc/collectd/conf-available/write-csv-stdout.conf \
    | tee -a /etc/collectd/collectd.conf

# Print resulting collectd.conf file, just to be sure.
RUN \
  nl /etc/collectd/collectd.conf
```

Build it by executing:
```
docker build -t mycollectd .
```
Launch it (interactively, to check if it works):
```
docker run --rm -it --name collectd --privileged --uts=host mycollectd
```

Running
-------
Running collectd inside a container can be tricky. When launching the container, some flags might be needed when invoking ```docker run```.

The reporting hostname in the collected metrics will be the container's hostname which will be confusing. To allow the container to find the host's real hostname, use the ```--uts=host``` flag. Notice that collectd will complain about the FQDN not being correctly configured. To solve this, you should include the ```global-FQDNLookup-disable.conf``` snippet in your configuration, as done in the example above.

Some plugins need access to host's ```/proc``` filesystem, which can be given by launching the container with the ```--privileged``` flag.

Warning: Not all Docker versions support the above flags. Check your version against the docs if some doesn't work.

Example run:
```
docker run -dit --restart=always --name collectd --uts=host <your-image-name>
```
