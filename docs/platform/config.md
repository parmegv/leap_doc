Configuration files
===================================

File format
-------------------------------------------

All configuration files are in JSON format. For example

    {
      "key1": "value1",
      "key2": "value2"
    }

Keys should match `/[a-z0-9_]/`

Unlike traditional JSON, comments are allowed. If the first non-whitespace character is '#' the line is treated as a comment.

    # this is a comment
    {
      # this is a comment
      "key": "value"  # this is an error
    }

Options in the configuration files might be nested hashes or arrays. For example:

    {
      "openvpn": {
        "ip_address": "1.1.1.1",
        "protocols": ["tcp", "udp"],
        "options": {
          "public_ip": false,
          "adblock": true
        }
      }
    }

If the value string is prefixed with an '=' character, the value is evaluated as ruby. For example:

    {
      "domain": {
        "public": "domain.org"
      }
      "api_domain": "= 'api.' + domain.public"
    }

In this case, "api_domain" will be set to "api.domain.org".

Inheritance
----------------------------------------

Suppose you have a node configuration for `bitmask/nodes/dns_europe.json` like so:

    {
       "services": "dns",
       "tags": ["production", "europe"],
       "ip_address": "1.1.1.1"
    }

This node will have hostname "dns-europe" and it will inherit from the following files (in this order):

    1. provider_base/common.json
    2. bitmask/common.json
    3. provider_base/services/dns.json
    4. bitmask/services/dns.json
    5. provider_base/tags/production.json
    6. bitmask/tags/production.json
    7. provider_base/tags/europe.json
    8. bitmask/tags/europe.json
    9. bitmask/nodes/dns_europe.json

The `provider_base` directory is under the `leap_platform` specified in the file `Leapfile`.

To see all the variables a node has inherited, you could run `leap inspect dns_europe`.

Macros
----------------------------------------

When using evaluated ruby in a JSON configuration file, there are several special macros that are available. These are evaluated in the context of a node (available as the variable `self`).

The following methods are available to the evaluated ruby:

`nodes`

  > A hash of all nodes. This list can be filtered.

`nodes_like_me`

  > A hash of nodes that have the same deployment tags as the current node (e.g. 'production' or 'local').

`global.services`

  > A hash of all services, e.g. `global.services['openvpn']`.

`global.tags`

  > A hash of all tags, e.g. `global.tags['production']`.

 `global.provider`

  > Can be used to access variables defined in `provider.json`, e.g. `global.provider.contacts.default`.

`file(filename)`

  > Inserts the full contents of the file. If the file is an erb template, it is rendered. The filename can either be one of the pre-defined file symbols, are it can be a path relative to the "files" directory in your provider instance. E.g, `file :ca_cert` or `files 'ca/ca.crt'`.

`file_path(filename)`

  >  Ensures that the file will get rsynced to the node as an individual file. The value returned by `file_path` is the full path where this file will ultimately live when deploy to the node. e.g. `file_path :ca_cert` or `file_path 'branding/images/logo.png'`.

`secret(:symbol)`

  > Returns the value of a secret in secrets.json (or creates it if necessary). E.g. `secret :couch_admin_password`

`variable.variable`

  > Any variable defined or inherited by a particular node configuration is available by just referencing it using either hash notation or object field notation (e.g. `['domain']['public']` or `domain.public`). Circular references are not allowed, but otherwise it is OK to nest evaluated values in other evaluated values. If a value has not been defined, the hash notation will return nil but the field notation will raise an exception.

Using hashes
-----------------------------------------

The macros `nodes`, `nodes_like_me`, `global.services`, and `global.tags` all return a hash of other objects. You can reference these hashes either directly or using a filter:

Direct access:

    nodes['vpn1']                # returns node named 'vpn1'
    global.services['openvpn']   # returns service named 'openvpn'

Filters:

    nodes[:public_dns => true] # all nodes where public_dns == true
    nodes[:services => 'openvpn', :services => 'tor'] # openvpn OR tor
    nodes[:services => 'openvpn'][:tags => 'production']  # openvpn AND production
