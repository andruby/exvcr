# ExVCR [![Build Status](https://secure.travis-ci.org/parroty/exvcr.png?branch=master "Build Status")](http://travis-ci.org/parroty/exvcr) [![Coverage Status](https://coveralls.io/repos/parroty/exvcr/badge.png?branch=master)](https://coveralls.io/r/parroty/exvcr?branch=master)


Record and replay HTTP interactions library for elixir.
It's inspired by Ruby's VCR (https://github.com/vcr/vcr), and trying to provide similar functionalities.

### Notes

- It only supports :ibrowse based HTTP interaction at the moment.
- HTTP interactions are recorded as JSON file.
    - The JSON file can be recorded automatically (vcr_cassettes) or manually updated (custom_cassettes)


### Usage
#### Code

```Elixir
defmodule ExVCR.MockTest do
  use ExUnit.Case
  import ExVCR.Mock

  setup_all do
    ExVCR.Config.cassette_library_dir("fixture/vcr_cassettes")
    :ok
  end

  test "example single request" do
    use_cassette "example_ibrowse" do
      :ibrowse.start
      {:ok, status_code, _headers, body} = :ibrowse.send_req('http://example.com', [], :get)
      assert status_code == '200'
      assert iolist_to_binary(body) =~ %r/Example Domain/
    end
  end

  test "httpotion" do
    use_cassette "example_httpotion" do
      assert HTTPotion.get("http://example.com", []).body =~ %r/Example Domain/
    end
  end
end
```

#### Custom Cassettes
Custom cassette can be defined in json format, by adding 2nd parameter of ExVCR.Config.cassette_library_dir method.

```Elixir
defmodule ExVCR.MockTest do
  use ExUnit.Case
  import ExVCR.Mock

  setup_all do
    ExVCR.Config.cassette_library_dir("fixture/vcr_cassettes", "fixture/custom_cassettes")
    :ok
  end
```

ExVCR uses url parameter to match request and cassettes. The "url" parameter in the json file is taken as regexp string.

**fixture/custom_cassettes/response_mocking.json**
```javascript
[
  {
    "request": {
      "url": "http://example.com"
    },
    "response": {
      "status_code": 200,
      "headers": {
        "Content-Type": "text/html"
      },
      "body": "<h1>Custom Response</h1>"
    }
  }
]
```