# flake-utils

**STATUS: alpha**

Pure Nix flake utility functions.

The goal of this project is to build a collection of pure Nix functions that don't
depend on nixpkgs, and that are useful in the context of writing other Nix
flakes.

## Usage

### `defaultSystems -> [<system>]`

A list of all the systems supported by the nixpkgs project.
Useful if you want add your platforms:

```nix
eachSystem (defaultSystems ++ ["armv7l-linux"]) (system: { hello = 42; })
```

### `eachSystem -> [<system>] -> (<system> -> attrs)`

A common case is to build the same structure for each system. Instead of
building the hierarchy manually or per prefix, iterate over each systems and
then re-build the hierarchy.

Eg:

```nix
eachSystem ["x86_64-linux"] (system: { hello = 42; })
# => { hello.x86_64-linux.hello = 42; }
```

### `eachDefaultSystem -> (<system> -> attrs)`

`eachSystem` pre-populated with `defaultSystems`.

### `mkApp { drv, name ? drv.pname or drv.name, execPath ? drv.passthru.execPath or "/bin/${name}"`

A small utility that builds the structure expected by the special `apps` and `defaultApp` prefixes.

## Example

Here is how it looks like in practice:

[$ example/flake.nix](example/flake.nix) as nix
```nix
{
  description = "Flake utils demo";

  inputs.flake-utils.url = "github:numtide/flake-utils";

  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system:
      let pkgs = nixpkgs.legacyPackages.${system}; in
      rec {
        packages.hello = pkgs.hello;
        defaultPackage = packages.hello;
        apps.hello = flake-utils.lib.mkApp { drv = packages.hello; };
        defaultApp = apps.hello;
      }
    );
}
```

## Known issues

```
$ nix flake check
warning: unknown flake output 'lib'
```

nixpkgs is currently having the same issue so I assume that it will be
eventually standardized.

