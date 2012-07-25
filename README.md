This is an experimental (unstable!) buildpack which installs
[Google Protobuf](http://code.google.com/p/protobuf/), and optionally
downloads .proto files from S3 and compiles them with `protoc`.

## Usage

```bash
# You must enable `user_env_compile` in every app which uses this
# buildpack, so that it can access your Heroku config. Note that this
# effectively means you are trusting your buildpack repo owners with
# your sensitive config.
$ heroku labs:enable user_env_compile

# Use the multi buildpack
$ heroku config:set BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git
$ cat .buildpacks
https://github.com/loku/heroku-buildpack-protobuf.git
https://github.com/heroku/heroku-buildpack-python.git
```

If you want to download proto files from S3 and compile them as part of the build:

```bash
# Add a .protos file to your app. The first line is a set of options
# to be passed to `protoc`. The remaining lines are paths to .proto
# files to download.
$ cat .protos
--python_out=protobufs
addressbook.proto

# You must also provide your AWS credentials and an S3 bucket from
# which to download protos
$ heroku config:set \
    AWS_ACCESS_KEY_ID=... \
    AWS_SECRET_ACCESS_KEY=... \
    PROTO_BUCKET=...
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

## Building protobuf

```bash
tar xzf protobuf-2.4.1.tar.gz
cd protobuf-2.4.1
vulcan build -c 'mkdir -p /app/.heroku/vendor && ./configure --prefix=/app/.heroku/vendor && make && make install' -n protobuf-2.4.1 -p /app/.heroku/vendor
```
