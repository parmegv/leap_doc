@title = 'Tests and Monitoring'
@summary = 'Testing and monitoring your infrastructure.'
@toc = true

## Tests

At any time, you can run troubleshooting tests on the nodes of your provider infrastructure to check to see if things seem to be working correctly. If there is a problem, these tests should help you narrow down precisely where the problem is.

To run tests on FILTER node list:

    leap test run FILTER

Alternately, you can run test on all nodes (probably only useful if you have pinned the environment):

    leap test

## Monitoring

If you have a node with the 'monitor' service, then this node will regularly poll every node to ask for the status of various health checks. These health checks include the checks run with `leap test`, plus many others.

You can log into the monitoring web interface via XXXX. The username and password are found in the secrets.json file in your provider directory.

TODO:

* add how to set up monitoring. just need to add 'monitor' service? what is it compatible with?
* add url
* add username/password
* add how to write your own `leap test` tests and/or nagios tests
