# Using Blazegraph as your triplestore

This project originally used Blazegraph instead of GraphDB. We changed because we wanted GraphDB's GeoSPARQL support and nice development console - but GraphDB is not open source, although a free version is available. If you wish to build a project that only has open-source software in it, you can use Blazegraph instead.

## Changes to the search code

## Changes to docker-compose and Helm charts

## Setting up Docker with Blazegraph
Assuming that you're starting from **this directory**:
1. To build and run the web app, Docker needs to know about some environment variables. There are examples ones in `dev.env` - copy it to `.env` and fill in the correct values for you. Save the file and then run `source .env`.
1. Install [Docker](https://docker.com)
   1. Also, For windows, click [here](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi) to install the WSL 2 (Windows Subsystem for Linux, version 2)
1. `cd docker`
1. `docker-compose up -d`
1. `docker-compose --profile setup up -d` in order to start all of the necessary services and set up Gleaner for indexing.
1. If you want to try queries out on Blazegraph directly, Go to your [local Blazegraph instance](http://localhost:9999/blazegraph/#namespaces) and be sure to click 'Use' next to the polder namespace.
NOTE: There is a missing step here. The crawled results need to be written to the triplestore. For now, you can run `./write-to-triplestore.sh`.
     1. For windows, you need to download [Cygwin](https://www.cygwin.com/setup-x86_64.exe).
     1. Change directory to the docker in Cygwin (`cd docker`).
     1. Run the `./write-to-triplestore.sh`. to write to triplestore.
1. Run the web app: `docker-compose --profile web up -d`

## Writing to Blazegraph
