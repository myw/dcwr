# DCWR - Docker Compose Wrap Run

Run a command inside Docker Compose as if it were local


## Documentation

Command-line usage:

```
dcwr [-h|--help] [-f|--file compose-file] [-q|--quiet] [-n|--no-run] [-k|--keep] [*]

  Transparently run a command inside a Docker Compose service

  dcwr runs the command inside a new Docker container for the Docker Compose
  service specified, as if it were running locally. It does this by taking
  every command argument that points to a file or directory, mounting it
  inside the Docker container and replacing it with the mount point
  destination.

  By default, it prints the final docker-compose command before running it.

Arguments:
  -h --help   - show this usage info
  -f --file   - path to docker-compose.yml file to use
                [defaults to ./docker-compose.yml]
  -q --quiet  - do not print the final command before running
  -n --no-run - do not run the command, just print it
  -k --keep   - do not add --rm to docker-compose run (default behavior)
  [*]         - arguments to docker-compose run
                [options, service name, command, etc]

Example:
  $ dcwr -n service-foo cmd-bar arg-baz README.md . -narf LICENSE /usr/share/man/man1/ls.1

  docker-compose run \
    --rm \
    --volume=/mydir/README.md:/tmp/9ZnUSxR1/4-README.md \
    --volume=/mydir/.:/tmp/9ZnUSxR1/5-. \
    --volume=/mydir/LICENSE:/tmp/9ZnUSxR1/7-LICENSE \
    --volume=/usr/share/man/man1/ls.1:/tmp/9ZnUSxR1/8-ls.1 \
    service-foo \
      cmd-bar \
        arg-bazbaz \
        /tmp/9ZnUSxR1/4-README.md \
        /tmp/9ZnUSxR1/5-. \
        -narf \
        /tmp/9ZnUSxR1/7-LICENSE \
        /tmp/9ZnUSxR1/8-ls.1
```


## Dependencies

- Docker
- Docker Compose
- `bash` >= 2.0
- Standard POSIX system utilities, specifically: `cat`, `env`, `mktemp`.


## Installation

Download or clone the repository and move, copy, or link `dcwr` into a directory on your `$PATH`, or
add the directory it's in to your `$PATH` through your shell's login environment settings. The best
practice on most systems is to copy `dcwr` into `$HOME/bin` if installing it for just a single
user, or `/usr/local/bin` if installing it for everyone.


## Caveats

This is a convenience utility and its approach fundamentally *cannot* account for all edge cases so
it has some important caveats:

- Arguments that are not strictly filenames, like `-f/foobar/baz`, or `--loc=path/to/file`, are not
  modified.
- New files/directories (i.e. that do not already exist on the host system) cannot be mounted this
  way: you will have to `touch` or `mkdir` them first on your own
- Commands that reference other files that are not specified in the container or otherwise
  mounted will not work without additional options
- Docker images that do not tolerate mounts into a temp directory will not work
- The executing command is always at the mercy of the image, so there are lots of other
  potential ways to "break" the expected behavior
