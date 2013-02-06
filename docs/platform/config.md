Configuration Files
=================================

All configuration files are in JSON format. For example

    {
      "key1": "value1",
      "key2": "value2"
    }

Keys should match /[a-z0-9_]/

Unlike traditional JSON, comments are allowed. If the first non-whitespace character is '#' the line is treated as a comment.

    # this is a comment
    {
      # this is a comment
      "key": "value"  # this is an error
    }

Options in the configuration files might be nested. For example:

    {
      "openvpn": {
        "ip_address": "1.1.1.1"
      }
    }

If the value string is prefixed with an '=' character, the value is evaluated as ruby. For example

    {
      "domain": {
        "public": "domain.org"
      }
      "api_domain": "= 'api.' + domain.public"
    }

In this case, "api_domain" will be set to "api.domain.org".

The following methods are available to the evaluated ruby:

* nodes -- A list of all nodes. This list can be filtered.
* global.services -- A list of all services.
* global.tags -- A list of all tags.
* file(filename) -- Inserts the full contents of the file. If the file is an erb
  template, it is rendered.
* file_path(filename) -- Ensures that the file will get rsynced to the node as an individual file. The value returned by `file_path` is where this file will utlimately live when on the node.
* secret(symbol) -- Returns the value of a secret in secrets.json (or creates it if necessary).
* variable -- Any variable inherited by a particular node is available by just referencing it using either hash notation or object notation (i.e. self['domain']['public'] or domain.public). Circular references are not allowed, but otherwise it is ok to nest evaluated values in other evaluated values.

