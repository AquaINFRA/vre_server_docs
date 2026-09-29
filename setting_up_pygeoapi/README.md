# Setting up pygeoapi

_Merret Buurman, IGB Berlin, 2026-09-29_


How to set up a pygeoapi instance for the AquaINFRA Virtual Research
Environment.

**Work in progress**


* Install pygeoapi according to pygeoapi's own documentation
  ([pygeoapi.io](https://pygeoapi.io))
* Make sure to enable asynchronous processing
* Make sure to go through the production checklist to run a proper setup, not
  the dev/testing setup
* Make sure the instance runs behind a reverse proxy that takes care of
  TLS termination
* [Add proper logging configuration](logging_processes.md)
* Optional: Log each processing job when it is started
* etc.

