# Open Source Routing Machine

The [Open Source Routing Machine](http://project-osrm.org/) is a high performance routing engine designed to run on OpenStreetMap data.

## Install OSRM on a server

### Building OSRM

First we install the dependencies : 
```
apt-get install git g++ cmake libboost-dev libboost-filesystem-dev libboost-thread-dev \
libboost-system-dev libboost-regex-dev libstxxl-dev libxml2-dev libsparsehash-dev libbz2-dev \
zlib1g-dev libzip-dev libgomp1 liblua5.1-0-dev \
libluabind-dev pkg-config libgdal-dev libboost-program-options-dev libboost-iostreams-dev \
libboost-test-dev libtbb-dev libexpat1-dev
```

And we get the backend sources from [OSRM Github repository](https://github.com/Project-OSRM/) : 
```
git clone https://github.com/Project-OSRM/osrm-backend.git /opt/osrm-backend
```

And launch the compilation :
```
cd /opt/osrm-backend
mkdir -p build
cd build
cmake ..
make
make install
```

### Running OSRM

In order to run OSRM, you need to feed it with OSM data, so you need to [download them on the server](./README.md) or use NFS to mount them from another server.

#### Extract routing data

When you have the data availables on your server, we need to configure the extract of OSRM.
OSM data represents a huge amount of entities, generally OSRM will need only a part of them depending of the way you plan to use it (car, cycle, pedestrian).

For configuring the entities to extract, we need to create a **profile.lua** file, you have predefined ones in **profiles** dir for bycicle, car and foot usages.

Note : It might be also necessary to add a symlink to the **profiles/lib** directory inside your build directory

For exaple you can use for car usage : 
```
cd /opt/osrm-backend/build
ln -s ../profiles/car.lua profile.lua
ln -s ../profiles/lib/
```

Now we have our profiles and we are ready to extract data from OSM ones, we have to think about syncing them with OSM datas daily update.

In order to do that, we add to the **/etc/cron.daily** script which do the OSM data update following script : 
```
/opt/osrm-backend/build/osrm-extract /opt/osm_data/auvergne.osm.pbf
/opt/osrm-backend/build/osrm-prepare /opt/osm_data/auvergne.osrm
```

And to allow OSRM reload them while running via Shared Memory we also add :
```
/opt/osrm-backend/build/osrm-datastore /opt/osm_data/auvergne.osrm
```

#### Add OSRM as startup service

In order to run OSRM as a service, we create a **/etc/init.d/osrm** file which contains : 
```
#!/bin/sh
# kFreeBSD do not accept scripts as interpreters, using #!/bin/sh and sourcing.
if [ true != "$INIT_D_SCRIPT_SOURCED" ] ; then
    set "$0" "$@"; INIT_D_SCRIPT_SOURCED=true . /lib/init/init-d-script
fi
### BEGIN INIT INFO
# Provides:          osrm
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Open Source Routing Machine
# Description:       Start the OSRM service for routing server
### END INIT INFO

# Author: Julien SABATIER <julien.sabatier@lepuyenvelay.fr>

DESC="Open Source Routing Machine"
DAEMON=/root/osrm-backend/build/osrm-routed
DAEMON_ARGS="-s"
OSRM_FILE="/opt/osm_data/auvergne.osrm"

do_start_cmd() {
        start-stop-daemon --start --quiet --background ${PIDFILE:+--pidfile ${PIDFILE}} \
            $START_ARGS \
            --startas $DAEMON --name $NAME --exec $DAEMON --test > /dev/null \
            || return 1
        /opt/osrm-backend/build/osrm-datastore $OSRM_FILE
        start-stop-daemon --start --quiet --background ${PIDFILE:+--pidfile ${PIDFILE}} \
            $START_ARGS \
            --startas $DAEMON --name $NAME --exec $DAEMON -- $DAEMON_ARGS \
            || return 2
}
```
Don't forget to replace _OSRM_FILE_ by your own osrm generated file.

Set this executable and add it to startup services : 
```
chmod +x /etc/init.d/osrm
update-rc.d osrm defaults
```

After what you can use this to start osrm : 
```
service osrm start
```

By default, OSRM is running on port 5000, so access it on _localhost:5000_.
