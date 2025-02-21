# go-torch (Windows update)

This repo includes minimal working version for running go-torch on Windows.
Additionally it embeds `flamegraph.pl`, `stackcollapse.pl` from: `https://github.com/brendangregg/FlameGraph` repository
and also required binaries and scripts from `strawberry perl` perl's interpreter portable version (license: `https://github.com/StrawberryPerl/Perl-Dist-Strawberry`).

Executing:

```bash
go run . pprof.out
```

where `pprof.out` is binary profile output from pprof should result in generating `torch.svg` flamegraph.

## Basic Usage

```bash
$ go-torch -h
Usage:
  go-torch [options] [binary] <profile source>

pprof Options:
  -u, --url=         Base URL of your Go program (default: http://localhost:8080)
  -s, --suffix=      URL path of pprof profile (default: /debug/pprof/profile)
  -b, --binaryinput= File path of previously saved binary profile. (binary profile is anything accepted by https://golang.org/cmd/pprof)
      --binaryname=  File path of the binary that the binaryinput is for, used for pprof inputs
  -t, --seconds=     Number of seconds to profile for (default: 30)
      --pprofArgs=   Extra arguments for pprof

Output Options:
  -f, --file=        Output file name (must be .svg) (default: torch.svg)
  -p, --print        Print the generated svg to stdout instead of writing to file
  -r, --raw          Print the raw call graph output to stdout instead of creating a flame graph; use with Brendan Gregg's flame graph perl script (see https://github.com/brendangregg/FlameGraph)
      --title=       Graph title to display in the output file (default: Flame Graph)
      --width=       Generated graph width (default: 1200)
      --hash         Colors are keyed by function name hash
      --colors=      Set color palette. Valid choices are: hot (default), mem, io, wakeup, chain, java,
                     js, perl, red, green, blue, aqua, yellow, purple, orange
      --hash         Graph colors are keyed by function name hash
      --cp           Graph use consistent palette (palette.map)
      --inverted     Icicle graph
Help Options:
  -h, --help         Show this help message
```

### Write flamegraph using /debug/pprof endpoint

The default options will hit `http://localhost:8080/debug/pprof/profile` for
a 30 second CPU profile, and write it out to torch.svg

```bash
$ go-torch
INFO[19:10:58] Run pprof command: go tool pprof -raw -seconds 30 http://localhost:8080/debug/pprof/profile
INFO[19:11:03] Writing svg to torch.svg
```

You can customize the base URL by using `-u`

```bash
$ go-torch -u http://my-service:8080/
INFO[19:10:58] Run pprof command: go tool pprof -raw -seconds 30 http://my-service:8080/debug/pprof/profile
INFO[19:11:03] Writing svg to torch.svg
```

Or change the number of seconds to profile using `--seconds`:

```bash
$ go-torch --seconds 5
INFO[19:10:58] Run pprof command: go tool pprof -raw -seconds 5 http://localhost:8080/debug/pprof/profile
INFO[19:11:03] Writing svg to torch.svg
```

### Using pprof arguments

`go-torch` will pass through arguments to `go tool pprof`, which lets you take
existing pprof commands and easily make them work with `go-torch`.

For example, after creating a CPU profile from a benchmark:

```bash
$ go test -bench . -cpuprofile=cpu.prof

# This creates a cpu.prof file, and the $PKG.test binary.
```

The same arguments that can be used with `go tool pprof` will also work
with `go-torch`:

```bash
$ go tool pprof main.test cpu.prof

# Same arguments work with go-torch
$ go-torch main.test cpu.prof
INFO[19:00:29] Run pprof command: go tool pprof -raw -seconds 30 main.test cpu.prof
INFO[19:00:29] Writing svg to torch.svg
```

Flags that are not handled by `go-torch` are passed through as well:

```bash
$ go-torch --alloc_objects main.test mem.prof
INFO[19:00:29] Run pprof command: go tool pprof -raw -seconds 30 --alloc_objects main.test mem.prof
INFO[19:00:29] Writing svg to torch.svg
```

## Integrating With Your Application

To add profiling endpoints in your application, follow the official
Go docs [here][].
If your application is already running a server on the DefaultServeMux,
just add this import to your application.

[here]: https://golang.org/pkg/net/http/pprof/

```go
import _ "net/http/pprof"
```

If your application is not using the DefaultServeMux, you can still easily
expose pprof endpoints by manually registering the net/http/pprof handlers or by
using a library like [this one](https://github.com/e-dard/netbug).

## Installation

```bash
go get github.com/patrulek/go-torch
```

You can also use go-torch using docker:

```bash
docker run uber/go-torch -u http://[address-of-host] -p > torch.svg
```

Using `-p` will print the SVG to standard out, which can then be redirected
to a file. This avoids mounting volumes to a container.

### Get the flame graph script

When using the `go-torch` binary locally, you will need the Flamegraph scripts
in your `PATH`:

```bash
cd $GOPATH/src/github.com/patrulek/go-torch
git clone https://github.com/brendangregg/FlameGraph.git
```

## Development and Testing

### Install the Go dependencies

```bash
go get github.com/Masterminds/glide
cd $GOPATH/src/github.com/patrulek/go-torch
glide install
```

### Run the Tests

```bash
$ go test ./...
ok    github.com/patrulek/go-torch   0.012s
ok    github.com/patrulek/go-torch/graph   0.017s
ok    github.com/patrulek/go-torch/visualization 0.052s
```
