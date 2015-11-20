# Open Source Routing Machine

The [Open Source Routing Machine](http://project-osrm.org/) is a high performance routing engine designed to run on OpenStreetMap data.

A fully usable OSRM server is composed of :

* [OSRM-backend](https://github.com/Project-OSRM/osrm-backend) : it implements an API to request OSM datas for routing datas
  * [Install it](./osrm-backend.md)
* [OSRM-frontend](https://github.com/Project-OSRM/osrm-frontend-v2) : it provides a graphical interface to query an OSRM server
  * [Install it](./osrm-frontend.md)

Of course you can install only the one you need, for exemple if you want to query the official OSRM server, you can just install the frontend part and configure the data provider.
