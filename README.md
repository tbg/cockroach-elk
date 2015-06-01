# Docker ELK stack for Cockroach logging

A fork with small changes on top of [fig-elk](https://github.com/deviantony/fig-elk)
for use with [Cockroach](https://github.com/cockroachdb/cockroach).

## Basic usage

Boot2docker prerequisites:
```
boot2docker poweroff
# map ports to localhost
for i in 5000 9200 7999 5601; do
 VBoxManage modifyvm "boot2docker-vm" --natpf1 "tcp-port$i,tcp,,$i,,$i";
done
boot2docker up
```

```
docker-compose up -d
./cockroach start --logjson [...] 2>&1 | netcat localhost 5000
# or (if you want to keep the log file)
./cockroach start --logjson [...] 2>&1 | tee log.log | netcat localhost 5000
# now use Kibana to look at the logs, on http://localhost:7999.
# once done,
docker-compose stop
# the elasticsearch data is in ./elasticsearch-data, so it is going to
# be available for the next session.
```

# HOW TO

## Setup

1. Install [Docker](http://docker.io).
2. Install [Docker-compose](http://docs.docker.com/compose/install/).
3. Clone this repository

### SELinux 

On distributions which have SELinux enabled out-of-the-box you will need to either re-context the files or set SELinux into Permissive mode in order for fig-elk to start properly. 
For example on Redhat and CentOS, the following will apply the proper context:

```
.-root@centos ~
`-$ chcon -R system_u:object_r:admin_home_t:s0 fig-elk/
```

## Usage

### Start the stack and inject logs

First step, you can edit the logstash-configuration in *logstash-conf/logstash.conf*. You can add filters you want to test for example.

Then, start the ELK stack using *docker-compose*:

```
$ docker-compose up
```

You can also choose to run it in background (detached mode):

```
$ docker-compose up -d
```

Now that the stack is running, you'll want to inject logs in it. The shipped logstash configuration allows you to send content via tcp:

```
$ nc localhost 5000 < /path/to/logfile.log
```


### Playing with the stack

The stack exposes 4 ports on your localhost:

* 5000: Logstash TCP input.
* 9200: Elasticsearch HTTP (with Marvel plugin accessible via [http://localhost:9200/_plugin/marvel](http://localhost:9200/_plugin/marvel))
* 7999: Kibana 3 web interface, access it via [http://localhost:7999](http://localhost:8080)
* 5601: Kibana 4 web interface, access it via [http://localhost:5601](http://localhost:5601)


### Boot2docker

If you're using *boot2docker*, you must access it via the *boot2docker* IP address:
* http://boot2docker-ip-address:9200/_plugin/marvel to access the Marvel plugin.
* http://boot2docker-ip-address:7999 to use Kibana 3.
* http://boot2docker-ip-address:5601 to use Kibana 4.
