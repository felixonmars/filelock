# Filelock [![Build Status][travis-img-url]][travis-url]

[travis-img-url]: https://travis-ci.org/sheerun/filelock.png
[travis-url]: https://travis-ci.org/sheerun/filelock

Heavily tested, but simple filelocking solution using [flock](http://linux.die.net/man/2/flock) command. It guarantees unlocking of files.

It works for sure on MRI 1.8, 1.9, 2.0, JRuby in both 1.8 and 1.9 mode, and Rubinius.

This gem doesn't support NFS. You can use it with [GlusterFS](http://www.gluster.org/), though.

## Basic Usage

```ruby
Filelock '/tmp/path/to/lock' do
  # do blocking operation
end
```

### Operation Timeout

You can also pass the timeout for blocking operation (default is 60 seconds):

```ruby
Filelock '/tmp/path/to/lock', :timeout => 10 do
  # do blocking operation
end
```

You can detect timeout by catching `Filelock::ExecTimeout`.

### Lock Acquiring Timeout

You can also pass a non-zero wait timeout for grabbing the lock (default is 1 day):

```ruby
Filelock '/tmp/path/to/lock', :wait => 3600 do
  # do blocking operation
end
```

You can detect this kind of timeout by catching `Filelock::WaitTimeout`.  Note that a wait timeout of 0 will block until the lock can be acquired instead of raising `Filelock::WaitTimeout`.

Note that lock file directory must already exist, and lock file is not removed after unlock.

### Getting handle to locked file


```ruby
Filelock '/tmp/path/to/lock' do |file|
  file.truncate
  file.write Process.pid
end
```

### Passing custom lock options

```ruby
# Non-blocking lock mode
Filelock '/tmp/path/to/lock', lock_options: File::LOCK_EX|File::LOCK_NB do
  # Do stuff.
end
```

### Passing custom file modes

```ruby
# Open filelock in append mode
Filelock '/tmp/path/to/lock', modes: 'a' do |file|
  # Do stuff.
end
```

## FAQ

*Does it support NFS?*

No. You can use more complex [lockfile](https://github.com/ahoward/lockfile) gem if you want to support NFS.

*The code is so short. Why shouln't I just copy-paste it?*

Because even such short code can have issues in future. File locking is very fragile operation. You may expect new releases of this gem fixing discovered bogus behavior (or introducing awesome features).

You are encouraged to use it if you develop gem that uses flock command, and care about running it on different ruby versions and platforms. Each has its own quirks with regard to flock command.

*How it's different from [lockfile](https://github.com/ahoward/lockfile) gem?*

Lockfile is filelocking solution handling NFS filesystems, based on homemade locking solution. Filelock uses [flock](http://linux.die.net/man/2/flock) UNIX command to handle filelocking on very low level. Also lockfile allows you to specify retry timeout. In case of Ruby's flock command this is hard-cored to 0.1 seconds.

*How it's different from [cleverua-lockfile](https://github.com/cleverua/lockfile) gem?*

Cleverua removes lockfile after unlocking it. Thas has been proven fatal both in my tests and in [filelocking advices from the Internet](http://world.std.com/~swmcd/steven/tech/flock.html). You could try find a way to remove lock file without breaking Filelock tests. I will be glad to accept such pull-request.

*What happens if my ruby process gets killed while holding a lock? Can new processes acquire this lock?*

Yes

## Contribute

Try to break Filelock in some way (note it doesn't support NFS).

## License

Filelock is MIT-licensed. You are awesome.
