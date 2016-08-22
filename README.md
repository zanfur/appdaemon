# Description

AppDaemon is a loosely coupled, multithreaded, sandboxed python execution environment for writing automation apps for [Home Assistant](https://home-assistant.io/) home automation software.

# Installation

Installation is either by pip3 or Docker.

## Clone the Repository

For either method you will need to clone the **AppDaemon** repository to the current local directory on your machine.

``` bash
$ git clone https://github.com/acockburn/appdaemon.git
```

Change your working directory to the repository root. Moving forward, we will be working from this directory.

``` bash
$ cd appdaemon
```

## Install using Docker

To build the Docker image run the following:

``` bash
$ docker build -t appdaemon .
```

## Install Using PIP3

Before running `AppDaemon` you will need to install the package:

```bash
$ sudo pip3 install .
```

# Configuration

When you have appdaemon installed by either method, copy the `conf/appdaemon.cfg.example` file to `conf/appdaemon.cfg`, then edit the `[AppDaemon]` section to reflect your environment:

```
[AppDaemon]
ha_url = <some_url>
ha_key = <some key>
logfile = STDOUT
errorfile = STDERR
app_dir = <Path to appdaemon dir>/conf/apps
threads = 10
latitude = <latitude>
longitude = <longitude>
elevation = <elevation
timezone = <timezone>
# Apps
[hello_world]
module = hello
class = HelloWorld
```

- `ha_url` is a reference to your home assistant installation and must include the correct port number and scheme (`http://` or `https://` as appropriate)
- `ha_key` should be set to your key if you have one, otherwise it can be removed.
- `logfile` is the path to where you want `AppDaemon` to keep its main log. When run from the command line this is not used - log messages come out on the terminal. When running as a daemon this is where the log information will go. In the example above I created a directory specifically for AppDaemon to run from, although there is no reason you can't keep it in the `appdaemon` directory of the cloned repository. If `logfile = STDOUT`, output will be sent to stdout instead of stderr when running in the foreground.
- `errorfile` is the name of the logfile for errors - this will usually be errors during compilation and execution of the apps. If `errorfile = STDERR` errors will be sent to stderr instead of a file.
- `app_dir` is the directory the apps are placed in
- `threads` - the number of dedicated worker threads to create for running the apps. Note, this will bear no resembelance to the number of apps you have, the threads are re-used and only active for as long as required to tun a particular callback or initialization, leave this set to 10 unless you experience thread starvation
- `latitude`, `longitude`, `elevation`, `timezone` - should all be copied from your home assistant configuration file

The `#Apps` section is the configuration for the Hello World program and should be left in place for initial testing but can be removed later if desired, as other Apps are added, App configuration is described in the [API doc](API.md).

## Docker

For Docker Configuration you need to take a couple of extra things into consideration.

Our Docker image is designed to load your configuration and apps from a volume at `/conf` so that you can manage them in your own git repository, or place them anywhere else on the system and map them using the Docker command line.

For example, if you have a local repository in `/Users/foo/ha-config` containing the following files:

```bash
$ git ls-files
configuration.yaml
customize.yaml
known_devices.yaml
appdaemon.cfg
apps
apps/magic.py
```

You will need to modify the `appdaemon.cfg` file to point to these apps in `/conf/apps`:

```
[AppDaemon]
ha_url = <some_url>
ha_key = <some key>
logfile = STDOUT
errorfile = STDERR
app_dir = /conf/apps
threads = 10
latitude = <latitude>
longitude = <longitude>
elevation = <elevation
timezone = <timezone>
```

You can run Docker and point the conf volume to that directory.

# Example Apps

There are a number of example apps under conf/examples, and the `conf/examples.cfg` file gives sample parameters for them.

# Running

As configured, AppDaemon comes with a single HelloWorld App that will send a greeting to the logfile to show that everything is working correctly.

## Docker

Assuming you have set the config up as described above for Docker, you can run it with the command:

```bash
$ docker run -d -v <Path to Config>/conf:/conf --name appdaemon appdaemon:latest
```

In the example above you would use:

```bash
$ docker run -d -v /Users/foo/ha-config:/conf --name appdaemon appdaemon:latest
```

Where you place the `conf` and `conf/apps` directory is up to you - it can be in downloaded repostory, or anywhere else on the host, as long as you use the correct mapping in the `docker run` command.

You can inspect the logs as follows:

```bash
$ docker logs appdaemon
2016-08-22 10:08:16,575 INFO Got initial state
2016-08-22 10:08:16,576 INFO Loading Module: /export/hass/appdaemon_test/conf/apps/hello.py
2016-08-22 10:08:16,578 INFO Loading Object hello_world using class HelloWorld from module hello
2016-08-22 10:08:16,580 INFO Hello from AppDaemon
2016-08-22 10:08:16,584 INFO You are now ready to run Apps!
```

Note that for Docker, the error and regular logs are combined.

## PIP3

You can then run AppDaemon from the command line as follows:

```bash
$ appdaemon conf/appdaemon.cfg
```

If all is well, you should see something like the following:

```
$ appdaemon conf/appdaemon.cfg
2016-08-22 10:08:16,575 INFO Got initial state
2016-08-22 10:08:16,576 INFO Loading Module: /export/hass/appdaemon_test/conf/apps/hello.py
2016-08-22 10:08:16,578 INFO Loading Object hello_world using class HelloWorld from module hello
2016-08-22 10:08:16,580 INFO Hello from AppDaemon
2016-08-22 10:08:16,584 INFO You are now ready to run Apps!
```

# AppDaemon arguments

usage: appdaemon [-h] [-d] [-p PIDFILE]
                 [-D {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
                 config

positional arguments:
  config                full path to config file

optional arguments:
  -h, --help            show this help message and exit
  -d, --daemon          run as a background process
  -p PIDFILE, --pidfile PIDFILE
                        full path to PID File
  -D {DEBUG,INFO,WARNING,ERROR,CRITICAL}, --debug {DEBUG,INFO,WARNING,ERROR,CRITICAL}
                        debug level

-d and -p are used by the init file to start the process as a daemon and are not required if running from the command line. 

-D can be used to increase the debug level for internal AppDaemon operations as well as apps using the logging function.

# Starting At Reboot
To run `AppDaemon` at reboot, I have provided a sample init script in the `./scripts` directory. These have been tested on a Raspberry PI - your mileage may vary on other systems.

# Operation

Since AppDaemon under the covers uses the exact same APIs as the frontend UI, you typically see it react at about the same time to a given event. Calling back to Home Assistant is also pretty fast especially if they are running on the same machine. In action, observed latency above the built in automation component is usually sub-second.

# Updating AppDaemon
To update AppDaemon after I have released new code, just run the following command to update your copy:

```bash
$ git pull origin
```
