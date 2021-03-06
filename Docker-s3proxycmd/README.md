Docker-s3proxycmd
=================

A standard tool that is used to work with S3 APIs is ```s3cmd```.  There are already containers built that expose the base functionality.  However, in some cases you may need to configure special parameters that cannot be read from environment variables directly or through run-time specifications.

For this we built the ```s3proxycmd``` Docker image.  This image does some work behind the scenes to get the environment variables specified into a config file that is read at run-time.  All you need to do is specify the appropriate variables and it does the rest.

### Docker Run
It is important to understand the networking requirements in order to talk with your S3 API.  In the example below we are running an S3 API on the same docker host, thus leveraging the ```--net container:769``` or ```container:containerid/name``` runtime parameter to ensure that our new container can talk to the existing container properly.  This may not be necessary for your setup.

In addition we are using the ```-v``` parameter at run-time to ensure that we map the local directory to the target.  This allows us to copy files that are on the container OS through the new S3 container to the S3 API target.

#### Fields

|field|
|-----|
|access_key|
|secret_key|
|proxy_host|
|proxy_port|
<br>

### List Buckets
In order to list buckets, simply fill in the proper parameters.  Any parameters can be left blank if desired.

```docker run -e 'access_key=wuser1@sanity.local' -e 'secret_key=+zjItiWiOVTrx00AiDRJOOvpFDQ+Hnwsn7aAJzsq' -e proxy_host=127.0.0.1 -e proxy_port=10101 -ti --net container:769 emccode/s3proxycmd ls```

### Help
Help is available by not specifying any command or ```--help```.  Actually, all ```s3cmd``` parameters are passed through.

```docker run -e 'access_key=wuser1@sanity.local' -e 'secret_key=+zjItiWiOVTrx00AiDRJOOvpFDQ+Hnwsn7aAJzsq' -e proxy_host=127.0.0.1 -e proxy_port=10101 -ti --net container:769 emccode/s3proxycmd```

### Put File
Create a blank random file at 1MB and send it to the S3 API.

The following command will create the file ```head -c 1M </dev/urandom >test.img```.  In order to copy the file you run this command, which maps the local directory and issues the copy command from the mounted directory
```docker run -e "access_key=wuser1@sanity.local" -e "secret_key=+zjItiWiOVTrx00AiDRJOOvpFDQ+Hnwsn7aAJzsq" -e proxy_host=127.0.0.1 -e proxy_port=10101 -ti --net container:769 -v $(pwd):/host emccode/s3proxycmd put /host/test.img s3://newbucket```.

### Interact with the container
All previous directions created a new container for every command. This process can create 'container sprawl'. To get to the console of the container, you must specify the ```--entrypoint``` flag.
```docker run -e 'access_key=wuser1@sanity.local' -e 'secret_key=+zjItiWiOVTrx00AiDRJOOvpFDQ+Hnwsn7aAJzsq' -e proxy_host=127.0.0.1 -e proxy_port=10101 -ti --entrypoint=/bin/bash  emccode/s3proxycmd```

Now the docker container is a long-running process. Running s3cmd must be prefaced with ```./bin/run_s3_proxy_cmd.sh```. For instance, listing the buckets is now done as ```./bin/run_s3_proxy_cmd.sh ls```. 