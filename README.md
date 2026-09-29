# AquaINFRA Virtual Research Environment (VRE) server documentation

This is a collection of documentation on how to set up the AquaINFRA VRE server.
This is the server that runs the various AquaINFRA processing tools as
pygeoapi processes behind an HTTP API of the OGC standard (Open Geospatial
Consortium). These tools can be accessed by any client via HTTP. In AquaINFRA,
the main frontend is the Galaxy platform, where the AquaINFRA tools are
accessible as Galaxy tools that send HTTP requests to this server in the
backend.

So this documentation covers:

* Setting up pygeoapi itself, behind a reverse proxy for TLS termination
* Setting up docker to run the tools
* Which static input data has to be present
* Deploying the various AquaINFRA tools
* Server to serve the static result files, and to serve some example files
* A cronjob to regularly remove old results
* TODO: Ansible playbook to automate deployment


## What's left...

AquaINFRA is a demo project and the setup of the AquaINFRA VRE is a test and
demo setup, which has grown over the years. Some implementation and architecture
choices would be taken differently in hindsight, but as this project was not
meant or funded to provide a fully matured production system, these
improvements were never done. For scaling up and supporting more load, a load
balancer that can lead the requests to several backend servers would be the
obvious next step (e.g. various pygeoapi instances, or distributing the docker
containers to several worker machines). Also, the tools themselves were mostly
not developed with scaling in mind, e.g. most of them are not prepared to use
several threads or cores.


## Funded by the European Union

This project has received funding from the European Commission’s Horizon Europe
Research and Innovation programme under grant agreement No 101094434. Project
coordinator: Aalborg Universitet (AAU). The information and views of this
website lie entirely with the authors. The European Commission is not
responsible for any use that may be made of the information it contains.
