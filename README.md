# purescript-typedenv [![Build](https://github.com/nsaunders/purescript-typedenv/workflows/CI/badge.svg)](https://github.com/nsaunders/purescript-typedenv/actions/workflows/ci.yml) [![Latest release](http://img.shields.io/github/release/nsaunders/purescript-typedenv.svg)](https://github.com/nsaunders/purescript-typedenv/releases) [![PureScript registry](https://img.shields.io/badge/dynamic/json?color=informational&label=registry&query=%24.typedenv.version&url=https%3A%2F%2Fraw.githubusercontent.com%2Fpurescript%2Fpackage-sets%2Fmaster%2Fpackages.json)](https://github.com/purescript/registry) [![purescript-typedenv on Pursuit](https://pursuit.purescript.org/packages/purescript-typedenv/badge)](https://pursuit.purescript.org/packages/purescript-typedenv)
## Type-directed environment parsing

<img src="https://github.com/nsaunders/purescript-typedenv/raw/master/meta/img/tile.png" alt="purescript-typedenv" align="right" />

The [`purescript-node-process` environment API](https://pursuit.purescript.org/packages/purescript-node-process/10.0.0/docs/Node.Process#v:getEnv)
provides environment variables in the form of an
[`Object String`](https://pursuit.purescript.org/packages/purescript-foreign-object/2.0.2/docs/Foreign.Object#t:Object)
(a string map), but it is left up to us to validate and to parse the values into some configuration model that can be used
safely throughout the rest of the program.

One of the more popular solutions would be something like this applicative-style lookup/validation/parsing into a record:

```purescript
type Config =
  { greeting :: String
  , count    :: Int
  }

parseConfig :: Object String -> Either String Config
parseConfig env =
  (\greeting count -> { greeting, count })
  <$> value "GREETING"
  <*> ( value "COUNT"
        >>= Int.fromString
        >>> Either.note "Invalid COUNT"
      )
  where
    value name =
      note ("Missing variable " <> name) $ lookup name env
```

However, this is a bit unsatisfying because the explicit lookups, parsing logic, and error handling are somewhat
verbose and might start to look like a lot of boilerplate as the `Config` model is extended with additional fields.
The value-level logic creates additional touchpoints when fields are added or removed, or their types change.

Instead, this library uses a type-directed approach, which starts with renaming the `Config` fields according to the
environment variable names from which their values are sourced:

```purescript
type Config =
  { "GREETING" :: String
  , "COUNT"    :: Int
  }
```

The `fromEnv` function now has enough information to convert the environment `Object String` to a typed record with
no need for explicit lookups, parsing, or error handling:

```purescript
parseConfig :: Object String -> Either String Config
parseConfig = Either.lmap printEnvError <<< TypedEnv.fromEnv (Proxy :: _ Config)
```

> **Note**
> An additional benefit not demonstrated here is that the `TypedEnv.fromEnv` function accumulates a list of
> errors, whereas the former example can only present one error at a time.

### Examples

To run one of the [examples](example), clone the repository and run the following command, replacing `<example-name>` with the name of the example.

```bash
spago -x example.dhall run -m Example.<example-name>
```

### Installation

via [spago](https://github.com/spacchetti/spago):
```bash
spago install typedenv
```

### Similar ideas
* [TypedEnv](https://github.com/freight-hub/TypedEnv)
* [dotenv-hs](https://github.com/stackbuilders/dotenv-hs) can validate variables against a `.schema.yml` file.
