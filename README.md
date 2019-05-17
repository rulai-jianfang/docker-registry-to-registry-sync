## Some problems fixed when We sync from private to private registry v2 sync up, better using the python file to test on local, then run the container later

## Problem 1
### Running via Docker:

```
docker run --rm -it \
    -v $(pwd)/config.yml:/config.yml \
    -v /var/run/docker.sock:/var/run/docker.sock
    stefanhudelmaier/docker-registry-to-registry-sync
```
### Change docker.sock permission and mount the docker.sock into container or it will incur 'file not found' error:
When run the python file in current environment directly, we need set /var/run/docker.sock's permission to 667 or 777 
When run with docker, then we need mount the /var/run/docker.sock into it
```
    -v /var/run/docker.sock:/var/run/docker.sock
```
error message:
```
sudo docker run --rm -it -v $(pwd)/config.yml:/config.yml stefanhudelmaier/docker-registry-to-registry-sync
Traceback (most recent call last):
  File "/usr/local/lib/python3.6/site-packages/urllib3/connectionpool.py", line 601, in urlopen
    chunked=chunked)
  File "/usr/local/lib/python3.6/site-packages/urllib3/connectionpool.py", line 357, in _make_request
    conn.request(method, url, **httplib_request_kw)
  File "/usr/local/lib/python3.6/http/client.py", line 1239, in request
    self._send_request(method, url, body, headers, encode_chunked)
  File "/usr/local/lib/python3.6/http/client.py", line 1285, in _send_request
    self.endheaders(body, encode_chunked=encode_chunked)
  File "/usr/local/lib/python3.6/http/client.py", line 1234, in endheaders
    self._send_output(message_body, encode_chunked=encode_chunked)
  File "/usr/local/lib/python3.6/http/client.py", line 1026, in _send_output
    self.send(msg)
  File "/usr/local/lib/python3.6/http/client.py", line 964, in send
    self.connect()
  File "/usr/local/lib/python3.6/site-packages/docker/transport/unixconn.py", line 46, in connect
    sock.connect(self.unix_socket)
FileNotFoundError: [Errno 2] No such file or directory

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/usr/local/lib/python3.6/site-packages/requests/adapters.py", line 440, in send
    timeout=timeout
  File "/usr/local/lib/python3.6/site-packages/urllib3/connectionpool.py", line 639, in urlopen
    _stacktrace=sys.exc_info()[2])
  File "/usr/local/lib/python3.6/site-packages/urllib3/util/retry.py", line 357, in increment
    raise six.reraise(type(error), error, _stacktrace)
  File "/usr/local/lib/python3.6/site-packages/urllib3/packages/six.py", line 685, in reraise
    raise value.with_traceback(tb)
  File "/usr/local/lib/python3.6/site-packages/urllib3/connectionpool.py", line 601, in urlopen
    chunked=chunked)
  File "/usr/local/lib/python3.6/site-packages/urllib3/connectionpool.py", line 357, in _make_request
    conn.request(method, url, **httplib_request_kw)
  File "/usr/local/lib/python3.6/http/client.py", line 1239, in request
    self._send_request(method, url, body, headers, encode_chunked)
  File "/usr/local/lib/python3.6/http/client.py", line 1285, in _send_request
    self.endheaders(body, encode_chunked=encode_chunked)
  File "/usr/local/lib/python3.6/http/client.py", line 1234, in endheaders
    self._send_output(message_body, encode_chunked=encode_chunked)
  File "/usr/local/lib/python3.6/http/client.py", line 1026, in _send_output
    self.send(msg)
  File "/usr/local/lib/python3.6/http/client.py", line 964, in send
    self.connect()
  File "/usr/local/lib/python3.6/site-packages/docker/transport/unixconn.py", line 46, in connect
    sock.connect(self.unix_socket)
urllib3.exceptions.ProtocolError: ('Connection aborted.', FileNotFoundError(2, 'No such file or directory'))

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "docker-registry-to-registry-sync.py", line 84, in <module>
    password=src_password)
  File "/usr/local/lib/python3.6/site-packages/docker/client.py", line 178, in login
    return self.api.login(*args, **kwargs)
  File "/usr/local/lib/python3.6/site-packages/docker/api/daemon.py", line 145, in login
    response = self._post_json(self._url('/auth'), data=req_data)
  File "/usr/local/lib/python3.6/site-packages/docker/api/client.py", line 250, in _post_json
    return self._post(url, data=json.dumps(data2), **kwargs)
  File "/usr/local/lib/python3.6/site-packages/docker/utils/decorators.py", line 46, in inner
    return f(self, *args, **kwargs)
  File "/usr/local/lib/python3.6/site-packages/docker/api/client.py", line 187, in _post
    return self.post(url, **self._set_request_timeout(kwargs))
  File "/usr/local/lib/python3.6/site-packages/requests/sessions.py", line 555, in post
    return self.request('POST', url, data=data, json=json, **kwargs)
  File "/usr/local/lib/python3.6/site-packages/requests/sessions.py", line 508, in request
    resp = self.send(prep, **send_kwargs)
  File "/usr/local/lib/python3.6/site-packages/requests/sessions.py", line 618, in send
    r = adapter.send(request, **kwargs)
  File "/usr/local/lib/python3.6/site-packages/requests/adapters.py", line 490, in send
    raise ConnectionError(err, request=request)
requests.exceptions.ConnectionError: ('Connection aborted.', FileNotFoundError(2, 'No such file or directory'))

```



## Problem 2
Need add dst user name and password to the config.xml, also the port number of the source and destination url
```
source_registry:
  url: https://source.xxx.com:5000
  username: xxxxx
  password: xxxxx


destination_registry:
  url: https://dst.xxx.com:5000
  username: yyyyy
  password: yyyyy
```





# docker-registry-to-registry-sync

Tool for syncing docker images from one registry to another

## Goal

Make is possible to sync all tags of a number of docker repositories
from one registry to another. Can be useful if you have an interal docker
registry but want to sync a subset of the docker images to a globally 
accessibly registry.

## Usage

### Configuration

Create a `config.yml` file. Example:

```yaml
source_registry:
  url: https://docker.example.com:5000
  username: some_user
  password: some_password

destination_registry:
  url: http://127.0.0.1:5000
    username: some_user
  password: some_password

repositories:
  - company/super-project
  - company/cool-project
```

The meaning of the settings:

* `source-registry`: The registry to sync from
* `destination-registry`: The registry to sync to
* `repositories`: A list of docker repositores, consisting of the repository name
  and image name (without the registry part).
  
When the images are synced, the registry part of the tag is replaced, e.g.
in the example above the tag `docker.example.com/company/super-project` is
re-tagged as `127.0.0.1:5000/company/super-project` and pushed to `127.0.0.1:5000`.

### Passwords as environment variables

You can specify the registry passwords using the `SOURCE_REGISTRY_PASSWORD` 
and `DESTINATION_REGISTRY_PASSWORD` environment variables instead of 
including them in the `config.yml` file.

### Running via Docker:

```
docker run --rm -it \
    -v $(pwd)/config.yml:/config.yml \
    -v /var/run/docker.sock:/var/run/docker.sock
    stefanhudelmaier/docker-registry-to-registry-sync
```
### Change docker.sock permission:
When run the python file in current environment directly, we need set /var/run/docker.sock's permission to 667 or 777 
When run with docker, then we need mount the /var/run/docker.sock into it
```
    -v /var/run/docker.sock:/var/run/docker.sock
```


When using environment variables for the passwords:

```
docker run --rm -it \
    -e SOURCE_REGISTRY_PASSWORD=secret \
    -e DESTINATION_REGISTRY_PASSWORD=secret \
    -v $(pwd)/config.yml:/config.yml \
    stefanhudelmaier/docker-registry-to-registry-sync
```

See [Image on Docker Hub](https://hub.docker.com/r/stefanhudelmaier/docker-registry-to-registry-sync/)

## Known limitations

* Does not work with Docker Hub due to a problem in the used registry client, see https://github.com/yodle/docker-registry-client/issues/42
