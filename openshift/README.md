# Running Puppet Server on Openshift

[Openshift](https://www.openshift.com/) is a container platform based on docker and kubernetes.

## Running the server

Create the project for puppet:

```
oc new-project puppetserver
```

The puppetserver image runs as root which openshift doesn't allow by default, so
we add the ability to do it for this project:

```
oc adm policy add-scc-to-user anyuid -z default
```

Create a new DeploymentConfig named `puppet` based on the
puppetserver-standalone image and expose it as a server on port 8140:

```
oc create deploymentconfig puppet --image=puppet/puppetserver-standalone
oc expose dc puppet --port 8140
```

## Running ephemeral agents

You can run some quick ephemeral agents inside the project to verify its working:

```
oc run centos-agent-$(uuidgen) --image puppet/puppet-agent-centos --restart=Never
oc run debian-agent-$(uuidgen) --image puppet/puppet-agent-debian --restart=Never
oc run ubuntu-agent-$(uuidgen) --image puppet/puppet-agent-ubuntu --restart=Never
```

If you run 'oc logs centos-agent-deadbeef1234` on the containers you created you
should see them checking in to the puppetserver. If you run `oc exec
puppet-1-s1llh -- find /etc/puppetlabs/puppet/ssl/ca/signed -type f` on the name
of your puppetserver pod, it should show all the signed certificates from the
client.
