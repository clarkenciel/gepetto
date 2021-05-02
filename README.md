# gepetto

An experiment in a headless chrome "pool" on kubernetes.

## Index

- [gepetto](#gepetto)
  - [Index](#index)
  - [Explain](#explain)
    - [Notes on seccomp](#notes-on-seccomp)
  - [Setup](#setup)
    - [Cluster](#cluster)

## Explain

I've worked a few places that needed to use headless chrome for non-testing
purposes: for web scraping, for printing views of our own web pages, etc.

At each of these places we wound up using [puppeteer](https://github.com/puppeteer/puppeteer)
to control a headless chrome browser, but we always took the easiest way out
and simply ran the browser as a subprocess of the main application or worker.

This meant that our application processes had memory footprints that were tied
to chrome and that the chrome instances were almost certainly being run with
too little security.

So, I've wondered for a while now what it might look like to run all of the
chrome instances on their own, with the right security and with the ability
to scale the chrome instances and manage their resource usage apart from
application services.

I'll be honest I thought this would be far more work intensive, but there's a
great project [here](https://github.com/Zenika/alpine-chrome) that has taken
care of the lift of building a good, light-weight docker image and the issues
and examples already provide a fair bit of knowledge for running this stuff
in kubernetes.

### Notes on seccomp

The safest way to run headless chrome containers is to use seccomp. This demo
includes a seccomp profile that is lightly adapted from
[Jess Frazelle's chrome seccomp profile](https://github.com/jessfraz/dotfiles/blob/master/etc/docker/seccomp/chrome.json).

Alas seccomp is a bit unwieldy to use on kubernetes: you need to make sure the
seccomp profiles you want to use are available on _nodes_ of your cluster. And
then, you need to either set configuration in your pod and/or container specs
_or_ set annotations, depending on the version of your cluster.
It's not that scary and the [kubernetes docs](https://kubernetes.io/docs/tutorials/clusters/seccomp/#create-pod-that-uses-the-container-runtime-default-seccomp-profile)
themselves are pretty clear.

Since we use kind for this demo we have the leisure of using cutting edge
kubernetes (more or less) so we get to use the spec-based settings, but be
warned that on older versions of kubernetes you may need to set annotations
instead.

## Setup

### Cluster

The following steps will get you set up with a locally running kubernetes cluster
via [kind](https://kind.sigs.k8s.io/).
It will create a new kubeconfig file at `.kubeconfig`.
This keeps us from interfering with any existing kubeconfig you may have, but
it _does_ mean you'll need to add `--kubeconfig=.kubeconfig` to your `kubectl`
commands (that said, you can always crete an alias as we do below :)).

1. Install kind
2. Install cluster

   ```bash
   kind create cluster --config ./cluster/config.yaml --kubeconfig .kubeconfig
   alias gkube='kubectl --kubeconfig=.kubeconfig'
   ```

3. Create work namespace

   ```bash
   gkube create ns gepetto
   gkube config set-context --current --namespace gepetto
   ```

4. Create Pool

   ```bash
   gkube apply -f k8s/pool.yml
   ```
