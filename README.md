# Versionary

Add versioning to your Elixir Plug and Phoenix built API's

[![Build Status](https://travis-ci.org/sticksnleaves/versionary.svg?branch=master)](https://travis-ci.org/sticksnleaves/versionary)
[![Coverage Status](https://coveralls.io/repos/github/sticksnleaves/versionary/badge.svg?branch=master)](https://coveralls.io/github/sticksnleaves/versionary?branch=master)

## Installation

The package can be installed by adding `versionary` to your list of dependencies
in `mix.exs`:

```elixir
def deps do
  [{:versionary, "~> 0.2.0"}]
end
```

## Usage

```elixir
def MyAPI.MyController do
  use MyAPI.Web, :controller

  plug Versionary.Plug.VerifyHeader, versions: ["application/vnd.app.v1+json"]

  plug Versionary.Plug.EnsureVersion, handler: MyAPI.MyErrorHandler
end
```

## MIME Support

Versionary can verify versions against media types configured within the
application by using `Versionary.Plug.VerifyHeader`'s `:accepts` option.

```elixir
config :mime, :types, %{
  "application/vnd.app.v1+json" => [:v1]
}
```

```elixir
def MyAPI.MyController do
  use MyAPI.Web, :controller

  plug Versionary.Plug.VerifyHeader, accepts: [:v1]

  plug Versionary.Plug.EnsureVersion, handler: MyAPI.MyErrorHandler
end
```

Please note that whenever you change media type configurations you must
recompile the `mime` library.

To force `mime` to recompile run `mix deps.clean --build mime`.

## Usage with Phoenix

### Simple example

Versionary is just a plug. That means Versionary works with Phoenix out of the
box. However, if you'd like Versionary to render a Phoenix error view when
verification fails use `Versionary.Plug.PhoenixErrorHandler`.

```elixir
defmodule MyAPI.Router do
  use MyAPI.Web, :router

  pipeline :api do
    plug Versionary.Plug.VerifyHeader, accepts: [:v1]

    plug Versionary.Plug.EnsureVersion, handler: Versionary.Plug.PhoenixErrorHandler
  end

  scope "/api", MyAPI do
    pipe_through :api

    get "/my_controllers", MyController, :index
  end

end
```

### Multiple API versions

All the version that are going to be used needs to be defined in the file
*config/config.exs*

```
# Previous code

config :mime, :types, %{
  "application/vnd.api.v1+json" => [:v1],
  "application/vnd.api.v2+json" => [:v2],
}

# Import environment specific config. This must remain at the bottom
# of this file so it overrides the configuration defined above.
import_config "#{Mix.env}.exs"
```

In this example **api** is the name of your application.

Then in the routes the pipeline that is going to be used for the api needs to
include both api mime types.

```elixir
defmodule MyAPI.Router do
  use MyAPI.Web, :router

  pipeline :api do
    plug Versionary.Plug.VerifyHeader, accepts: [:v1, :v2]

    plug Versionary.Plug.EnsureVersion, handler: Versionary.Plug.PhoenixErrorHandler
  end

  scope "/api", MyAPI do
    pipe_through :api

    get "/my_controllers", MyController, :index
  end

end
```

Finally in the controller a way to differenciate between both versions needs to
be defined, see the following example

```elixir
defmodule MyAPI.MyController do
  use MyAPI, :controller

  def index(%{private: %{version: [:v1]}} = conn, _params) do
    conn
    |> render("index.v1.json", %{})
  end

  def index(%{private: %{version: [:v2]}} = conn, _params) do
    conn
    |> render("index.v2.json", %{})
  end
end
```

Then just define methods to support both versions on the view.

## Plug API

### [Versionary.Plug.VerifyHeader](https://hexdocs.pm/versionary/Versionary.Plug.VerifyHeader.html)

Verify that the version passed in to the request as a header is valid. If the
version is not valid then the request will be flagged.

This plug will not handle an invalid version.

#### Options

`accepts` - a list of strings or atoms representing versions registered as
MIME types. If at least one of the registered versions is valid then the
request is considered valid.

`versions` - a list of strings representing valid versions. If at least one of
the provided versions is valid then the request is considered valid.

`header` - the header used to provide the requested version (Default: `Accept`)

### [Versionary.Plug.EnsureVersion](https://hexdocs.pm/versionary/Versionary.Plug.EnsureVersion.html)

Checks to see if the request has been flagged with a valid version. If the
version is valid, the request continues, otherwise the request will halt and the
handler will be called to process the request.

#### Options

`handler` - the module used to handle a request with an invalid version
(Default: [Versionary.Plug.ErrorHandler](https://hexdocs.pm/versionary/Versionary.Plug.ErrorHandler.html))

### [Versionary.Plug.Handler](https://hexdocs.pm/versionary/Versionary.Plug.Handler.html)

Behaviour for handling requests with invalid versions. You can create your own
custom handler with this behaviour.

# Development

## Run tests

Before running the test make sure all dependencies are installed, to do that just
run the following command

```bash
$ mix deps.get
```

Then to run the test run this command

```bash
$ mix tests
```
