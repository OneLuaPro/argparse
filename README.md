# argparse with CMake Support

argparse is a feature-rich command line parser for Lua inspired by argparse for Python. argparse supports positional arguments, options, flags, optional arguments, subcommands and more. argparse automatically generates usage, help, and error messages, and can generate shell completion scripts.

## Example

Simple example:

```lua
-- script.lua
local argparse = require "argparse"

local parser = argparse("script", "An example.")
parser:argument("input", "Input file.")
parser:option("-o --output", "Output file.", "a.out")
parser:option("-I --include", "Include locations."):count("*")

local args = parser:parse()
```

`args` contents depending on command line arguments:

```bash
$ lua script.lua foo
```

```lua
{
   input = "foo",
   output = "a.out",
   include = {}
}
```

```bash
$ lua script.lua foo -I/usr/local/include -Isrc -o bar
```

```lua
{
   input = "foo",
   output = "bar",
   include = {"/usr/local/include", "src"}
}
```

Error messages depending on command line arguments:

```bash
$ lua script.lua foo bar
```

```
Usage: script [-h] [-o <output>] [-I <include>] <input>

Error: too many arguments
```

```bash
$ lua script.lua --help
```

```
Usage: script [-h] [-o <output>] [-I <include>] <input>

An example. 

Arguments: 
   input                 Input file.

Options: 
   -h, --help            Show this help message and exit.
   -o <output>, --output <output>
                         Output file. (default: a.out)
   -I <include>, --include <include>
                         Include locations.
```

```bash
$ lua script.lua foo --outptu=bar
```

```
Usage: script [-h] [-o <output>] [-I <include>] <input>

Error: unknown option '--outptu'
Did you mean '--output'?
```

## Installing argparse

### Prerequisites

argparse with CMake Support relies on an installation of [Lua with CMake Support](https://github.com/KritzelKratzel/lua#readme). The same toolchain which has been used with *Lua with CMake Support* is required. argparse gets all necessary path information from CMake `liblua` package data and will be installed automatically into the right directory locations - no hassle with `package.path` settings anymore.

### Install

**Note**: As argparse does not comprise any files to be compiled and linked (just files to be installed properly) the mention of architecture or configuration below is somewhat arbitrary. 

Open `Developer Command Prompt for VS 2022` and change drive and directory. Download and unpack sources or simply clone this repository:

```cmd
c:
cd c:\Temp
git clone git@github.com:KritzelKratzel/argparse.git
cd argparse
```

CMake strongly encourages out-of-source builds.

```cmd
mkdir build
cd build
cmake .. -G "Visual Studio 17 2022" -A <arch>
cmake --build . --config Release
cmake --install . --config Release
```

Replace `<arch>` with your desired architecture. Available architectures with selected `Visual Studio 17 2022` generator are `Win32`, `x64`, `ARM` and `ARM64`. argparse documentation is available in `<lua_install_dir>/share/doc/argparse` after install. In addition, a tutorial is available [online](http://argparse.readthedocs.org). 

## Sync this fork with original argparse repository

Open Git Bash and execute `./SyncFork.sh`.

```bash
John Doe@DESKTOP-1HK25HF MINGW64 /c/misc/argparse (master)
$ ./SyncFork.sh
Original remote repo found.
Already on 'master'
Your branch is up to date with 'origin/master'.
From github.com:KritzelKratzel/argparse
 * branch            master     -> FETCH_HEAD
Already up to date.
Already up to date.
Everything up-to-date
John Doe@DESKTOP-1HK25HF MINGW64 /c/misc/argparse (master)

argparse comes with a testing suite located in `spec` directory. [busted](https://github.com/lunarmodules/busted) is required for testing, it can be installed using LuaRocks. Run the tests using `busted` command from the argparse folder.

## License

argparse is licensed under the same terms as Lua itself (MIT license).
