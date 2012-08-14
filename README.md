This is an experimental (unstable!) buildpack which installs [Google
Protobuf](http://code.google.com/p/protobuf/).

## Usage

```bash
# Use the multi buildpack
$ heroku config:set BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git
$ cat .buildpacks
https://github.com/loku/heroku-buildpack-protobuf.git
https://github.com/heroku/heroku-buildpack-python.git
```

If your app needs a Protobuf installation at runtime (e.g. for
node-protobuf):

```bash
# This will install Protobuf with a prefix /app/.heroku/vendor
$ heroku config:set VENDOR_PROTOBUF=1
# For node-protobuf you currently also need to set:
$ heroku config:set LD_LIBRARY_PATH=/app/.heroku/vendor/lib \
    PKG_CONFIG_PATH=/app/.heroku/vendor/lib/pkgconfig
```

I am not sure whether `VENDOR_PROTOBUF` will work for anything except
node-protobuf (e.g. the Python CPP implementation). Java and pure-Python,
of course, don't need `VENDOR_PROTOBUF`.

If you need to configure this buildpack, e.g. setting
`VENDOR_PROTOBUF` or `PROTOBUF_TARBALL_URL`, you must enable the
user_env_compile feature for the app:

```bash
$ heroku labs:enable user_env_compile
```


## Building your own protobuf

Protobuf is installed from a tarball downloaded, by default, from
<https://protobuf.s3.amazonaws.com/protobuf-2.4.1.tgz>. To change this
URL, set `PROTOBUF_TARBALL_URL`. `If-None-Match: <md5sum>` is relied
upon to avoid redownloading the tarball on every build (S3 supports
this. Note that currently the tarball must be publicly available).

`PROTOBUF_TARBALL_URL` should point to a tarball built as follows,
using [vulcan](https://github.com/heroku/vulcan/):

```bash
tar xzf protobuf-2.4.1.tar.gz
cd protobuf-2.4.1
vulcan build -c 'mkdir -p /app/.heroku/vendor && ./configure --prefix=/app/.heroku/vendor && make && make install' -n protobuf-2.4.1 -p /app/.heroku/vendor
```
