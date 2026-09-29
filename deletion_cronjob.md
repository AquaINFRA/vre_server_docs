# Regular deletion of process outputs (cronjob)

_Merret Buurman, IGB Berlin, 2026-09-29_

The tools that run on the AquaINFRA VRE server produce output files.
In order to not fill up the disk, they need to be cleaned regularly.
For this, every hour cronjobs are run that find, list, and delete
all output files and directories that are older than a certain age
threshold. The deleted files and directories are listed in a log
for verification.

_TODO: We could go down to every night._

## Cronjob definition

These cronjobs are defined in `/etc/crontab`. The results to be deleted
are in `/var/www/nginx/download/out`. The listings of the deleted files
are written to 

Here are the relevant parts of the cronjob definition (with a 90-day
threshold):

```
SHELL=/bin/sh
NOWTODAY=date +%Y%m%d

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
...
# Pygeoapi results
50 *    * * *   root    echo $(date)" In a minute, will delete the following "$(find /var/www/nginx/download/out -type f -mtime +90 | wc -l)" old files ("$(date)")" >> /opt/remove_old_outputs_$(${NOWTODAY}).log
51 *    * * *   root    find /var/www/nginx/download/out -type f -mtime +90 -exec ls -lpah -- '{}' \; >> /opt/remove_old_outputs_$(${NOWTODAY}).log
52 *    * * *   root    echo $(date)" Deleting..." >> /opt/remove_old_outputs_$(${NOWTODAY}).log
53 *    * * *   root    find /var/www/nginx/download/out -type f -mtime +90 -print -delete >> /opt/remove_old_outputs_$(${NOWTODAY}).log
56 *    * * *   root    echo $(date)" In a minute, will delete the following "$(find /var/www/nginx/download/out -type d -empty -mtime +90 | wc -l)" old directories, if empty ("$(date)")" >> /opt/remove_old_outputs_$(${NOWTODAY}).log
57 *    * * *   root    find /var/www/nginx/download/out -type d -empty -mtime +90 -exec ls -ldpah -- '{}' \; >> /opt/remove_old_outputs_$(${NOWTODAY}).log
58 *    * * *   root    echo $(date)" Deleting..." >> /opt/remove_old_outputs_$(${NOWTODAY}).log
59 *    * * *   root    find /var/www/nginx/download/out -type d -empty -mtime +90 -print -delete >> /opt/remove_old_outputs_$(${NOWTODAY}).log
...

```


## What the commands do

* This just announces the number of files (using `find ... -type f`) older than 90 days (using `-mtime +90`) that will be deleted (counted by `wc -l`):

```
echo $(date)" In a minute, will delete the following "$(find /var/www/nginx/download/out -type f -mtime +90 | wc -l)" old files ("$(date)")" >> /opt/remove_old_outputs_$(${NOWTODAY}).log
```

* This prints a list of these files (listed by `ls -lpah`):

```
find /var/www/nginx/download/out -type f -mtime +90 -exec ls -lpah -- '{}' \; >> /opt/remove_old_outputs_$(${NOWTODAY}).log
```

* This just announces the actual deletion:

```
echo $(date)" Deleting..." >> /opt/remove_old_outputs_$(${NOWTODAY}).log
```

* This runs the actual deletion (using `find ... -delete`), while also confirming each deletion (using `-print`):

```
find /var/www/nginx/download/out -type f -mtime +90 -print -delete >> /opt/remove_old_outputs_$(${NOWTODAY}).log
```

* And then the same for (empty) directories (using `find ... -type d -empty`) ...


## Example log output

The log will contain lines like this:

```
Mon Sep 28 03:50:01 UTC 2026 In a minute, will delete the following 30 old files (Mon Sep 28 03:50:01 UTC 2026)
-rw-r--r-- 1 root root 687 Jun 29 03:15 /var/www/nginx/download/out/map-shapefile-points/job_a8aa9359-7368-11f1-a84c-fa163e42fba0/interactive_map_a8aa9359-7368-11f1-a84c-fa163e42fba0_files/leafletfix-1.0.0/leafletfix.css
-rw-r--r-- 1 root root 68 Jun 29 03:15 /var/www/nginx/download/out/map-shapefile-points/job_a8aa9359-7368-11f1-a84c-fa163e42fba0/interactive_map_a8aa9359-7368-11f1-a84c-fa163e42fba0_files/rstudio_leaflet-1.3.1/images/1px.png
-rw-r--r-- 1 root root 1.1K Jun 29 03:15 /var/www/nginx/download/out/map-shapefile-points/job_a8aa9359-7368-11f1-a84c-fa163e42fba0/interactive_map_a8aa9359-7368-11f1-a84c-fa163e42fba0_files/rstudio_leaflet-1.3.1/rstudio_leaflet.css
...
Mon Sep 28 03:52:01 UTC 2026 Deleting...
/var/www/nginx/download/out/map-shapefile-points/job_a8aa9359-7368-11f1-a84c-fa163e42fba0/interactive_map_a8aa9359-7368-11f1-a84c-fa163e42fba0_files/leafletfix-1.0.0/leafletfix.css
/var/www/nginx/download/out/map-shapefile-points/job_a8aa9359-7368-11f1-a84c-fa163e42fba0/interactive_map_a8aa9359-7368-11f1-a84c-fa163e42fba0_files/rstudio_leaflet-1.3.1/images/1px.png
/var/www/nginx/download/out/map-shapefile-points/job_a8aa9359-7368-11f1-a84c-fa163e42fba0/interactive_map_a8aa9359-7368-11f1-a84c-fa163e42fba0_files/rstudio_leaflet-1.3.1/rstudio_leaflet.css
...
Mon Sep 28 09:56:01 UTC 2026 In a minute, will delete the following 0 old directories, if empty (Mon Sep 28 09:56:01 UTC 2026)
Mon Sep 28 09:58:01 UTC 2026 Deleting...
Mon Sep 28 10:50:01 UTC 2026 In a minute, will delete the following 0 old files (Mon Sep 28 10:50:01 UTC 2026)
Mon Sep 28 10:52:01 UTC 2026 Deleting...
...
Mon Sep 28 10:56:01 UTC 2026 In a minute, will delete the following 2 old directories, if empty (Mon Sep 28 10:56:01 UTC 2026)
drwxr-xr-x 2 ubuntu ubuntu 4.0K Jun 29 10:25 /var/www/nginx/download/out/netcdf-logger-extract/job_e8c8fc3c-73a4-11f1-9b13-fa163e42fba0/
drwxr-xr-x 2 ubuntu ubuntu 4.0K Jun 29 10:32 /var/www/nginx/download/out/netcdf-logger-extract/job_c3ec25c3-73a5-11f1-86db-fa163e42fba0/
Mon Sep 28 10:58:01 UTC 2026 Deleting...
/var/www/nginx/download/out/netcdf-logger-extract/job_e8c8fc3c-73a4-11f1-9b13-fa163e42fba0
/var/www/nginx/download/out/netcdf-logger-extract/job_c3ec25c3-73a5-11f1-86db-fa163e42fba0
...
```
