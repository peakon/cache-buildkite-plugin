# Cache Buildkite Plugin

A [Buildkite plugin](https://buildkite.com/docs/agent/v3/plugins) to restore and save
directories by cache keys. For example, use the checksum of a `.resolved` or `.lock` file
to restore/save built dependencies between independent builds, not just jobs.

With tarball or rsync, if source folder has changes, this will not fail your build, instead will surpress and continue.

For S3, Instead of sync millions of files, I just tarball before S3 operation then copy to s3. This will reduce both time and cost on AWS.

Plus, In addition to tarball & rsync, we also do not re-create another tarball for same cache key if its already exists.

## Restore & Save Caches

```yml
steps:
  - plugins:
    - gencer/cache#v2.0.1:
        cache_key: "v1-cache-{{ checksum 'Podfile.lock' }}"
        paths: [ "Pods/", "Rome/" ]
```

## Cache Key Templates

The cache key is a string, which support a crude template system. Currently `checksum` is
the only command supported for now. It can be used as in the example above. In this case
the cache key will be determined by executing a _checksum_ (actually `sha1sum`) on the
`Podfile.lock` file, prepended with `v1-cache-`.

## S3 Storage (Using tarball)

This plugin uses AWS S3 cp to cache the paths into a bucket as defined by environment
variables defined in your agent.

```bash
export BUILDKITE_PLUGIN_CACHE_S3_BUCKET_NAME="my-unique-s3-bucket-name"
export BUILDKITE_PLUGIN_CACHE_S3_PROFILE="my-s3-profile" # optional
```

The paths are synced using Amazon S3 into your bucket using a structure of
`organization-slug/pipeline-slug/cache_key.tar`, as determined by the Buildkite environment
variables.

## Rsync Storage

You can also use rsync to store your files using the ``rsync_storage`` config parameter.
If this is set it will be used as the destination parameter of a ``rsync -az`` command.

```yml
steps:
  - plugins:
    - gencer/cache#v2.0.1:
        rsync_storage: '/tmp/buildkite-cache'
        cache_key: "v1-cache-{{ checksum 'Podfile.lock' }}"
        paths: [ "Pods/", "Rome/" ]
```

The paths are synced using `rsync_storage/cache_key/path`. This is useful for maintaining a local
cache directory, even though this cache is not shared between servers, it can be reused by different
agents/builds.

## Tarball Storage

You can also use tarballs to store your files using the ``tarball_storage`` config parameter.
If this is set it will be used as the destination parameter of a ``tar -cf`` command.

```yml
steps:
  - plugins:
    - gencer/cache#v2.0.1:
        tarball_storage: '/tmp/buildkite-cache'
        tarball_keep_max_days: 7 # Optional. Removes tarballs older than 7 days.
        cache_key: "v1-cache-{{ checksum 'Podfile.lock' }}"
        paths: [ "Pods/", "Rome/" ]
```

The paths are synced using `tarball_storage/cache_key.tar`. This is useful for maintaining a local
cache directory, even though this cache is not shared between servers, it can be reused by different
agents/builds.


Original work by [@danthorpe]( https://github.com/danthorpe/cache-buildkite-plugin)

Copyright (C) 2020 Gencer W. Genç.
Licensed as **MIT**.
