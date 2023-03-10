# Contributing

## Overview

This documents explains the processes and practices recommended for contributing enhancements to
this operator.

- Generally, before developing enhancements to this charm, you should consider [opening an issue
  ](https://github.com/canonical/mysql-k8s-operator/issues) explaining your use case.
- If you would like to chat with us about your use-cases or proposed implementation, you can reach
  us at [Canonical Mattermost public channel](https://chat.charmhub.io/charmhub/channels/charm-dev)
  or [Discourse](https://discourse.charmhub.io/).
- Familiarising yourself with the [Charmed Operator Framework](https://juju.is/docs/sdk) library
  will help you a lot when working on new features or bug fixes.
- All enhancements require review before being merged. Code review typically examines
  - code quality
  - test coverage
  - user experience for Juju administrators of this charm.
- Please help us out in ensuring easy to review branches by rebasing your pull request branch onto
  the `main` branch. This also avoids merge commits and creates a linear Git commit history.

## Developing

You can create an environment for development with `tox`:

```shell
tox devenv -e integration
source venv/bin/activate
```

### Testing

```shell
tox run -e format          # update your code according to linting rules
tox run -e lint            # code style
tox run -e unit            # unit tests
tox run -e integration-*   # integration tests
tox                        # runs 'lint' and 'unit' environments
```

## Build charm

Build the charm in this git repository using:

```shell
charmcraft pack
```

### Deploy

```bash
# Create a model
juju add-model dev
# Enable DEBUG logging
juju model-config logging-config="<root>=INFO;unit=DEBUG"
# Deploy the charm
juju deploy ./mysql-k8s_ubuntu-20.04-amd64.charm \
    --resource mysql-image=ubuntu/mysql
```

## Canonical Contributor Agreement

Canonical welcomes contributions to the MySQL Operator. Please check out our [contributor agreement](https://ubuntu.com/legal/contributors) if you're interested in contributing to the solution.

## Appendix

### Charm lifecycle flowcharts

```mermaid
flowchart TD
    id101([leader_elected]) --> id102[generate\npassword/configs]
    id102 --> id103[store peer databag]
    id103 --> id104[add `configured`\nflag]
    id104 --> id105((return))

    id201([pebble_ready]) --> id202{if not `configured` \nnor peer relation}
    id202 --> id203>defer]
    id202 -- else --> id204[add pebble layer]
    id204 --> id205[configure users]
    id205 --> id206[configure instance]
    id206 --> id207{is leader?}
    id207 -- no --> id208((return))
    id207 -- yes --> id209[create cluster]
    id209 --> id208

    id301([peer_relation_joined]) --> id302{if not `configured`}
    id302 --> id303>defer]
    id302 -- else --> id304{is leader?}
    id304 -- no --> id311
    id304 -- yes --> id305{container\ncannot connect\nor pebble layer\nnot running}
    id305 --> id303
    id305 -- no --> id306[check instance configuration]
    id306 --> id307{instance configured?}
    id307 -- no --> id303
    id307 -- yes --> id308{any units in state transfer?}
    id308 -- yes --> id303
    id308 -- no --> id309[store instance address to databag]
    id309 --> id310[add instance to cluster]
    id310 --> id311((return))
```

