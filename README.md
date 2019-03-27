# Resto

Resto is an utility for sending out HTTP requests according to a YAML definition. YAML is used to represent definition of the request, this request is then executed and the JSON response is then reencoded to YAML and outputted to the terminal.

## Installation

Currently only installation from source is supported.

```shell
$ git clone https://github.com/adamruzicka/resto
$ cd resto
$ rm *.gem
$ gem build resto.gemspec
$ gem install ./resto-*.gem
```

## Usage

1) The base is having a yaml file describing how to perform the request. At bare minimum it needs to contain `host` key.

```yaml
# resto.yml
---
host: https://archive.org/metadata/principleofrelat00eins
```

2) Run it
```shell
$ resto -f resto.yml
---
created: 1553646003
d1: ia600205.us.archive.org
d2: ia800205.us.archive.org
dir: "/19/items/principleofrelat00eins"
files:
- name: __ia_thumb.jpg
  source: original
  mtime: '1548683866'
  size: '16910'
  md5: 772476cfceffb70fc55be52300bf18a6
  crc32: c9231f5c
  sha1: dd9d4033cb5bfeb3a1c2f432edc545d6293edaab
  format: Item Tile
  rotation: '0'
----- B< ----- SNIPPED ----- B< -----
```

## Definition file

As mentioned previously, the definition file needs to contain at least the `host` key. Some more can be set. The template is considered to be an ERB template and is rendered. This allows us to pass CLI arguments into the definition.

```yaml
# resto.yml
---
host: http://something.somewhere.com/user/<%= env['id'] %>
```

```shell
resto -f resto.yml -e id=15
```

This would make a `GET` request to `http://something.somewhere.com/user/15`.

### method

This represents the HTTP method for the outgoing request, currently supported ones are `GET` and `POST`. If undefined, `GET` will be used by default.

### host

The host to reach out to, can contain the path.

### path

The path on the host to hit, is used to construct the URL to talk to.

### headers

A hash containing headers to set on the request.

```yaml
headers:
  Content-Type: application/json
  User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:12.0) Gecko/20100101 Firefox/12.0
```

### payload

For `POST` this determines the payload sent out as the body of the request.

```yaml
payload:
  title: My new post
  content: |
    This is my new post describing how to use resto
```

### parameters

A hash describing which parameters are sent out.

```yaml
parameters:
  per_page: 10
  page: 5
```

### custom attributes

Optionally custom attributes can be used and later selected using the `-A` switch. Let's consider we have a host, where we want to hit `/foos` and `/bars`. Both these require authorization. Resto provides a neat-enough solution for this.

```yaml
# resto.yml
template: &template
  host: http://something.somewhere.com
  headers:
    Content-Type: application/json
    Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

foos:
  <<: *template
  path: /foos

bars:
  <<: *template
  path: /bars
  method: POST
  payload:
    name: "BAR!"
```

Then you can invoke a request to either `/foos` by executing `resto -f resto.yml -A foos` or `/bars` by calling `resto -f resto.yml -A bars`.

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/adamruzicka/resto.
