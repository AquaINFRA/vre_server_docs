# Log every process

_Merret Buurman, IGB Berlin, 2026-09-29_

To get an easy overview over each process that is started on pygeoapi, you can
add a custom logger that logs only that.

Note: This is likely not up to date, as this was first done on an older
version of pygeoapi

## Add logger definition

At the top of the file `pygeoapi/process/manager/base.py` (beneath the other
Logger definition line `LOGGER = logging.getLogger(__name__)`), add the
definition of a Logger object called `PLOGGER`:

```
PLOGGER = logging.getLogger("my_process_logger")
PLOGGER.setLevel(logging.INFO)
PLOGGER.propagate = False
```

The _propagate_ means that any lines logged to this logger will not be added
to the main log file.

Also import these:

```
from datetime import datetime
from zoneinfo import ZoneInfo
```

## Add log lines

Further below, in the function `execute_process`, add this snippet:

```
        ts_now = datetime.now(ZoneInfo("Europe/Berlin"))
        date = ts_now.strftime("%Y-%m-%d")
        time = ts_now.strftime("%H:%M:%S")
        PLOGGER.info(f'{process_id};{date};{job_id};{time}')
```

## Add to logconfig

Now we want those logged lines to be written into a log file.

So we add to the pygeoapi's `log_config.json`:

* ... this logger:

```
        "my_process_logger": {
            "level": "DEBUG",
            "handlers": ["prochandler"],
            "propagate": false
        }
```

* ... this handler:

```
        "prochandler": {
            "class": "logging.handlers.RotatingFileHandler",
            "level": "DEBUG",
            "filename": "/opt/.../logs/pygeoapi-proc.log",
            "maxBytes": 10485760,
            "backupCount": 40,
            "encoding": "utf8",
            "formatter": "supersimple"
        }
```


* ... and this formatter:

```
"supersimple": {
    "format": "%(message)s"
}
```

## Activate

For the added python lines to take effect, you'll have to reinstall/restart
pygeoapi.

Then (after running a process), the log will contain lines similar to this:

```
mitgcm-prep;2026-09-25;0c0de2a6-b87d-11f1-8c1d-fa163e42fba0;03:04:23
```
