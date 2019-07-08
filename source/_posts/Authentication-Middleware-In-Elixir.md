---
title: Authentication Middleware In Elixir
date: 2019-07-03 18:44:34
category: Programming
tags: 
  - Elixir
  - Functional
---
One of the first projects I was working on in Elixir is an API gateway. Like everyone else, I saw Pheonix, which is a cool framework for building web servers which is similar to Express.js, but, for my use case, I wanted raw performance for the API gateway, and one of its features was to have basic authentication as well as bearer authentication for json web tokens. One way to achieve this was using Plugs, which are built in the language. A plug is similar to a middleware in Express.js, it accepts input, does some manipulation and either halts the request or passes it on. In this post I will show how I implemented an authentication plug in Elixir.


## Router with plugs
Lets first look at how plugs look in a cowboy based router.
```elixir
defmodule Gateway.MainRouter do
  use Plug.Router
  use Plug.ErrorHandler

  import Plug.Conn

  plug(:match)
  plug(:fetch_query_params)
  plug(Plug.RequestId)
  plug(:dispatch)

  match _ do
   conn
    |> put_resp_content_type("application/json")
    |> send_resp(200, "Found")
  end
end

```

We have our use statements as well as our import for Plug.Conn. Lines 7,8,9 and 10 are plugs. Every request that comes in passes through each of the plugs before it gets to the route matches. This is useful because we want to build an authentication plug that we can reuse later on and that can be plugged wherever.

## How to build a plug
Lets first see how to build a plug, from the plug documentation:
```elixir
defmodule MyPlug do
  import Plug.Conn

  def init(options) do
    options
  end

  def call(conn, _opts) do
    conn
    |> put_resp_content_type("text/plain")
    |> send_resp(200, "Hello world")
  end
end
```

Each plug must have 2 methods, one is init, which allows passing options at compile time to the plug and the other, the call method with the connection received. The call method must either halt the request or return the conn object.

## What do we need for authorization
Looks pretty simple, now lets think about the authorization we need before we start writing it.
  - We must accept basic auth for server to server communcation
  - We must accept jwt(bearer) auth for client to server communication
  - If no auth header return 401

## Defining our 401
Before we start, lets define our 401, if its hit we halt and return 401:
```elixir
  defmodule Plug.Auth do
    defp send_401(
      conn,
      data \\ %{message: "Please make sure you have authentication header"}
    ) do
      conn
      |> put_resp_content_type("application/json")
      |> send_resp(401, Poison.encode!(data))
      |> halt
    end
  end
```

## Defining the call method
Now that we took care of the not authorized lets see how we implement the auth itself. The call method will extract auth header and call authenticate:

```elixir
  def call(%Plug.Conn{request_path: _path} = conn, _opts) do
    conn
    |> get_auth_header
    |> authenticate
  end
```

## Getting the authorization header
We try get the auth header and call authenticate. If you ask where is the if/else/try/catch/send_401, Elixir has pattern matching and guard clauses we can leverage to avoid all the boilerplate of defensive programming to keep our focus on our use case.
Lets define the get_auth_header method:
```elixir
  defmodule Plug.Auth do
    defp get_auth_header(conn) do
      case get_req_header(conn, "authorization") do
        [token] -> {conn, token}
        _ -> {conn}
      end
    end
  end
```

The `get_req_header` is from the Plug.Conn and allows getting the authorization header. We will return a tuple with the token if it exists or a tuple with the conn itself.

# Authenticating
Now, the interesting part, after `get_auth_header` we call authenticate. Lets leverage Elixir's power to extract what we need:
```elixir
defmodule Plug.Auth do
  @secret "my super secret"
  @alg "HS256"
  @signer Joken.Signer.create(@alg, @secret)

  defp authenticate({conn, "Bearer " <> jwt}) do
    case Joken.verify(jwt, @signer) do
      {:ok, claims} -> assign(conn, :user, claims)
      {:error, err} -> send_401(conn, %{error: err})
    end
  end

  defp authenticate({conn, "Basic " <> token}) do
    [username, password] =
      token
      |> Base.decode64!(padding: false)
      |> String.split(":")

    case Cache.get("users:#{username}") do
      nil ->
        send_401(conn, "User does not exist")

      %User{name: username, password: salted_password} ->
        case Bcrypt.verify_pass(password, salted_password) do
          true -> assign(conn, :user, %{name: username})
          false -> send_401(conn, "Password is incorrect")
        end
    end
  end

  defp authenticate(_) do
    send_401(conn)
  end
end
```

We defined 3 authenticate methods. If the `get_auth_header` returned a tuple with only conn in it, it means we have no auth header, so we call send_401, any other header returned that does not contain basic or bearer receives 401 as well. The other two methods are either Basic or Bearer. Elixir allows us to pattern match on binary strings, because our auth strings always start with Basic or Bearer we can match on them using the `start_of_string <> rest_of_string` syntax. If we hit the bearer auth, we check that the jwt matches, if the basic auth is hit, we compare it to what we have in our cache/db. If auth passes, the assign username/claims is called on the conn object and then returns it.

## Connecting it all together
Now, lets go back to our initial router and add our plug to it:
```elixir
defmodule Gateway.MainRouter do
  use Plug.Router
  use Plug.ErrorHandler

  import Plug.Conn

  plug(:match)
  plug(Plug.Auth) <- our auth plug
  plug(:fetch_query_params)
  plug(Plug.RequestId)
  plug(:dispatch)

  match _ do
   conn
    |> put_resp_content_type("application/json")
    |> send_resp(200, "Found")
  end
end
```
Now we have an authenticated app!

## Full example
Here is the full module, dont forget to add Joken or some other jwt checking library and replace Cache for basic auth with your own implementation:
```elixir
defmodule Plug.Auth do
  import Plug.Conn

  @secret "my super secret"
  @alg "HS256"
  @signer Joken.Signer.create(@alg, @secret)

  def init(opts) do
    opts
  end

  defp authenticate({conn, "Bearer " <> jwt}) do
    case Joken.verify(jwt, @signer) do
      {:ok, claims} -> assign(conn, :user, claims)
      {:error, err} -> send_401(conn, %{error: err})
    end
  end

  defp authenticate({conn, "Basic " <> token}) do
    [username, password] =
      token
      |> Base.decode64!(padding: false)
      |> String.split(":")

    case Cache.get("users:#{username}") do
      nil ->
        send_401(conn, "User does not exist")

      %User{name: username, password: salted_password} ->
        case Bcrypt.verify_pass(password, salted_password) do
          true -> assign(conn, :user, %{name: username})
          false -> send_401(conn, "Password is incorrect")
        end
    end
  end

  defp authenticate(_) do
    send_401(conn)
  end

  defp send_401(
         conn,
         data \\ %{message: "Please make sure you have authentication header"}
       ) do
    conn
    |> put_resp_content_type("application/json")
    |> send_resp(401, Poison.encode!(data))
    |> halt
  end

  defp get_auth_header(conn) do
    case get_req_header(conn, "authorization") do
      [token] -> {conn, token}
      _ -> {conn}
    end
  end

  def call(%Plug.Conn{request_path: _path} = conn, _opts) do
    conn
    |> get_auth_header
    |> authenticate
  end
end
```

## Summary
We learned what are plugs in Elixir, later on we saw how to implement a plug ourselves. Afterwards, we defined our authentication requirements and last, we implemented the authentication, leveraging Elixir's pattern matching capabilities.