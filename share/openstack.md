# openstack

`openstack` is the command-line client for an OpenStack cloud. It talks to the APIs, mostly Keystone for identity, Nova for servers, Glance for images, Neutron for networks, and Cinder for volumes. It does not manage local files, and it does not talk to the hypervisor on this machine. With no subcommand it prints help and exits non-zero. A subcommand that succeeds prints a table, or nothing, and exits 0. A rejected API call prints a message on standard error and exits non-zero. GNU/Linux with python-openstackclient is assumed here. The same client runs on macOS. BusyBox is irrelevant. Older clouds need extra plugin packages before `network`, `volume`, or `loadbalancer` commands exist. Identity v3 is assumed. A project is not a user. A name is not an id.

Basic form: openstack server list

Global options come before the subcommand. `server list` is the subcommand. Authentication is not a flag you can skip. It comes from `--os-cloud`, from `OS_*` variables, or from a prompt. A name that collides with a flag needs care. The client does not expand shell globs for you. Quote a name that contains spaces. `--` is not how this client ends options. Put global options first.

## Show the servers you can see
Also asked as: openstack server list; list instances; list vms; nova list; what is running
`server list` prints the servers in the current project. The table has an id, a name, a status, and the networks and addresses the client could look up. It does not print every project. An admin passes `--all-projects` for that. A server in `ERROR` or `BUILD` is still a row. The command does not start anything, and it does not refresh until you run it again.

```sh
openstack server list
```

People run it, see nothing, and think the cloud is empty. The project in the cloud config is the usual reason. `server list --all-projects` as a normal user fails or stays empty. A deleted server is gone from this list. A shelved server may still be listed, with a status that is not `ACTIVE`. The name column is not unique. The id is.

## Log in to a cloud
Also asked as: openstack login; OS_AUTH_URL; clouds.yaml; --os-cloud; authenticate; keystone
The client authenticates before the subcommand. `--os-cloud name` selects a cloud from `clouds.yaml`. The file is `~/.config/openstack/clouds.yaml`, or `/etc/openstack/clouds.yaml`. Environment variables such as `OS_AUTH_URL`, `OS_USERNAME`, `OS_PASSWORD`, `OS_PROJECT_NAME`, `OS_USER_DOMAIN_NAME`, and `OS_PROJECT_DOMAIN_NAME` override it. Identity v3 needs the domain names. A password in the environment shows up in the process list.

```sh
openstack --os-cloud cloud server list
```

People export `OS_TENANT_NAME` and leave the project empty. That is the v2 name. v3 wants `OS_PROJECT_NAME` and a project domain. A wrong password and a wrong URL can look the same in a short error. `--debug` prints the request. Do not paste that log. It contains the token. `openstack token issue` is the small command that only checks the login. A catalog with no compute endpoint fails later, on `server list`, not at token issue.

## Pick columns and machine-readable output
Also asked as: openstack -f json; -f value; -c ID; openstack format; parse openstack; csv output
`-f` chooses the format. `table` is the default. `json` and `yaml` are the structured forms. `value` prints the columns you named with `-c` and no header. `-c` can be repeated. These are global options. They go before the subcommand. `value` is the form to capture in a shell variable. `json` is the form to hand to a parser.

```sh
openstack -f value -c ID server list
```

People parse the table with `awk` and the columns shift when a network is added. `-f value -c ID -c Name` is the stable cut. A command that prints a single object, `server show`, uses the same `-f` and `-c`. `-c` names the field the API returned, not the header you guessed. `openstack server show -f json -- ID` shows the real keys. `value` with no `-c` is often empty or useless. Name the columns.

## Show one server
Also asked as: openstack server show; describe an instance; server details; vm status; fault message
`server show` prints one server. The argument is a name or an id. An id is the safe one, because names are not unique. The output includes flavor, image, status, addresses, and a fault field when the server is in `ERROR`. It does not print the console log. That is `console log show`. It does not print the password. The cloud does not keep the admin password from `server create` unless you set one up to retrieve.

```sh
openstack server show -- server
```

People pass a name that matches two servers and the client errors, or worse, acts on one. Use the id from `server list`. A `BUILD` server has no address yet. Showing it again is how you wait, or pass `--wait` on the create. `server show` does not start a stopped server. Status `SHUTOFF` is real. The flavor in the output is the flavor now, not a promise you can resize without `server resize`.

## Create a server
Also asked as: openstack server create; boot an instance; nova boot; create a vm; launch from an image
`server create` asks Nova to boot a server. You give a name, an image, a flavor, and a network. `--image`, `--flavor`, and `--network` take a name or an id. `--key-name` injects an existing keypair. The command returns when the API accepts the server, which can be before the server is `ACTIVE`. `--wait` waits for `ACTIVE` or a failure. The new id is in the output.

```sh
openstack server create --image image --flavor flavor --network network --key-name key server
```

People forget `--network` on a cloud that has no default network, and the server goes to `ERROR` with a fault about ports. People pass a local filename as `--image`. The image must already be in Glance. `image create` uploads one. A name collision does not replace the old server. You get a second server with the same name. The admin password in the create output is shown once if the cloud returns it. Store it then. It is not in `server show` later.

## Delete a server
Also asked as: openstack server delete; destroy a vm; terminate an instance; nova delete; remove a server
`server delete` asks Nova to delete each server you name. It does not ask you to confirm. The row disappears from `server list` when the delete is accepted or finished. Volumes attached as non-boot volumes are not deleted with the server unless the volume's delete-on-termination flag says so. A floating IP is not released. A name that matches nothing is an error.

```sh
openstack server delete -- server
```

People delete the server and expect the volume and the floating IP to go too. They remain, and the project still pays for them. Delete those by id after you have copied the ids down. `server delete` of a `BUILD` server is legal and races the build. There is no trash. `--wait` on delete waits until it is gone. Without it, a following command can still see the server in `deleting`.

## List images
Also asked as: openstack image list; glance image-list; available images; public images; private images
`image list` prints images the project can see. Public images and this project's private images are both there. `--private` and `--public` filter. The id is what `server create --image` should use when names collide. An image in `queued` or `saving` is not bootable yet. The command does not download the bytes.

```sh
openstack image list
```

People look for an image they just uploaded and forget the visibility. A private image in another project is invisible. A shared image appears only after the share is accepted, on clouds that require acceptance. The name is not unique. `image show` prints size, disk format, and status. `active` is the status you want before a boot.

## Upload an image
Also asked as: openstack image create; upload a qcow2; glance image-create; import an image; create a private image
`image create` registers an image and, with `--file`, uploads the bytes. `--disk-format` is `qcow2` or `raw` or another format Glance knows. `--container-format` is almost always `bare`. `--private` keeps it in this project. The file is a local path. The command reads it and pushes it. A large file takes as long as the upload takes.

```sh
openstack image create --disk-format qcow2 --container-format bare --file file image
```

People name the format wrong and the image goes `active` anyway, then the boot fails in the hypervisor. The format has to match the file. `--file` is not a URL. A remote image is a different import path, and many clouds disable it. The create output's id is the image. The name you gave is not reserved. A second create makes a second image. `--progress` is the GNU-client flag that shows the upload. Without it, a quiet client is still uploading.

## List flavors
Also asked as: openstack flavor list; instance sizes; nova flavor-list; vcpus ram disk; flavor id
`flavor list` prints the sizes the project may boot. A flavor has vCPUs, RAM in megabytes, and a root disk in gigabytes. `server create --flavor` takes the name or the id. A flavor with a disk of 0 often means the image size is the disk. A flavor the project is not allowed to use is absent, not marked secret.

```sh
openstack flavor list
```

People pick a flavor whose disk is smaller than the image and the server goes `ERROR`. Compare `image show` disk size with the flavor disk. A public flavor and a private flavor can share a name. Pass the id. `flavor list --all` is how an admin sees disabled flavors. A normal user does not. The flavor is a Nova object. Changing it later is `server resize`, not an edit of the row.

## List networks
Also asked as: openstack network list; neutron net-list; which network to boot on; private network; external network
`network list` prints networks this project can see. `--external` keeps the networks that can provide floating IPs. A tenant network is where the server port is created. The external network is where the floating IP is allocated. `server create --network` wants the tenant network, not the external one, on a normal cloud.

```sh
openstack network list
```

People boot on the external network and the cloud rejects the port, or they boot with no `--network` and Nova has no default. `network show` prints the subnets. A network with no subnet cannot give the server an address. Shared networks appear in the list even though another project owns them. The owner field is the check. Names such as `private` are not special to the client. They are only names.

## Create a floating IP and attach it
Also asked as: openstack floating ip create; add a public ip; server add floating ip; associate floating ip; neutron floatingip
A floating IP is allocated from an external network, then attached to a server. `floating ip create` takes the external network. `server add floating ip` takes the server and the address. They are two calls. Creating one does not attach it. The server also needs a port on a network that can route to that external network.

```sh
openstack floating ip create network
```

```sh
openstack server add floating ip server floating-ip
```

People run the create and look at `server show` for the address. It is not there until the add. People add an address from another project and get an error. The address is an object with an owner. `floating ip list` shows which are down, meaning unattached. An attached address still exists after the server is deleted, and it still counts against quota. `floating ip delete` releases it.

## Open a security group
Also asked as: openstack security group rule create; allow ssh; open a port; security group; icmp and tcp
A new project has a default security group. It often allows egress and denies ingress. `security group rule create` adds one rule. `--ingress`, `--protocol tcp`, `--dst-port 22`, and `--remote-ip 0.0.0.0/0` opens SSH from anywhere. The group must be on the server. `server create --security-group` sets that. A later change is `server add security group`.

```sh
openstack security group rule create --ingress --protocol tcp --dst-port 22 --remote-ip 0.0.0.0/0 default
```

People open the port in the group and the server is in another group. The packet is still dropped. `server show` lists the groups. People also open the group and forget the subnet's router has no gateway. Security groups are not routes. `--remote-ip` is a CIDR. `0.0.0.0/0` is every IPv4 address. ICMP is `--protocol icmp`. A rule does not print a confirmation beyond the new rule object. `security group rule list` is the check.

## Create a volume and attach it
Also asked as: openstack volume create; cinder create; attach a disk; server add volume; extra disk
`volume create` allocates a Cinder volume. `--size` is gigabytes. `--bootable` marks it bootable. It is not attached to a server until `server add volume`. The device name you ask for is a hint. The guest may see another name. A volume in `creating` or `available` is not on the server yet. `in-use` means attached.

```sh
openstack volume create --size 10 volume
```

```sh
openstack server add volume server volume
```

People create the volume and look for a new disk in the guest. The attach is the second command, and the guest may need a rescan. Deleting the server does not delete the volume by default. `volume delete` fails while it is `in-use`. Detach first with `server remove volume`. A volume and a server in different availability zones often fail the attach. The zone is on `volume show` and `server show`.

## Download a console log or a console URL
Also asked as: openstack console log show; boot log; openstack console url show; vnc url; why the server will not boot
`console log show` prints the server's console output, the boot messages Nova captured. `--lines` limits it. `console url show` prints a URL for VNC or serial, depending on `--novnc` or the cloud default. The URL is often short-lived. Neither command is SSH. SSH needs the guest, the key, the security group, and a route.

```sh
openstack console log show -- server
```

People treat an empty console log as "the server has no console." The cloud may disable it, or the guest has not written yet. A URL that fails to load is often an expired token or a VNC address your network cannot reach. The client printed the URL the API returned. It did not open a tunnel. `server show` status `ERROR` plus `console log show` is the pair to read before you boot another copy.

## Use a keypair
Also asked as: openstack keypair create; ssh key; import a public key; keypair list; nova keypair-add
`keypair create --public-key file` imports a public key you already have. Without `--public-key`, the client generates a key and prints the private key once. That private key is not stored in the cloud. `server create --key-name` injects the public key for cloud-init, or for the image's account. The guest username is a property of the image, not of the keypair command.

```sh
openstack keypair create --public-key file key
```

People generate a key, miss the private half, and cannot log in. Redirect that one create, or import a key you already hold. A keypair name is per user, in the project. A server booted before the key existed does not gain it. Rebuild, or install the key in the guest. `keypair list` does not print private keys. The cloud never had them, except in the one response to a generate.

## Show quotas
Also asked as: openstack quota show; why the create failed; instance quota; cores ram volumes; over limit
`quota show` prints the project's compute quotas when you pass the project, and the defaults vary by service. `quota show --compute`, `--volume`, and `--network` pick a service. A create that fails with "over limit" is this table. The numbers are limits, not usage. Usage is `server list` plus the quota, or the service's usage command where the cloud has one.

```sh
openstack quota show --compute project
```

People raise a quota in their head by deleting a row in `ERROR` and the count does not drop. A server in `ERROR` still consumes an instance until it is deleted. A volume in `error` still consumes gigabytes. `--network` quotas are Neutron's. Floating IPs have their own line. A normal user cannot raise the number. `quota set` is an admin command, and it is ignored on a cloud that takes quotas from outside OpenStack.

## See why a command was denied
Also asked as: openstack --debug; 403 forbidden; 401 unauthorized; failed to discover; catalog; policy denied
A 401 is authentication. The password, the user domain, or the URL is wrong. A 403 is authorization. The token worked and the policy said no. `--debug` prints the HTTP status and the URL. It also prints the token. Do not send that output to a ticket without redacting it. `catalog list` shows which services the token can see. A missing compute endpoint is a catalog problem, not a server problem.

```sh
openstack catalog list
```

People rerun the same command and expect a different 403. Policy does not change because you asked twice. An admin command such as `--all-projects` or `quota set` needs an admin role in that project. A cloud config pointed at the wrong region returns an endpoint that then refuses the call. `--os-region-name` and the region in `clouds.yaml` are the things to compare. The client exit status is non-zero for all of these. The status does not say 401 versus 403. The message does.
