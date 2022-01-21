<h2> Building a cost-effective Kubernetes cluster - (Krish and Marcus) </h2>
<h3>Notes for students that help us</h3>

What we need:
- A laptop (serving as the master node), preferably running OSX or Linux
- Multiple single board computers (we are using Raspberry Pi's running Raspbian Linux since it is optimised for the pis themselves)
- HDMI Cables
- Power cords

Connect to nodes over SSH or through plugging it into a monitor (Repeat the following for each Pi)

```ssh pi@ipaddress```


```
sudo systemctl --now enable sshd # start and enable SSH on reboot
```

For each pi, upgrade it and install vim

```
sudo apt update && sudo apt upgrade
sudo apt install vim
```
The images on the pis are unfortunately 32bit. Just edit ``/boot/config.txt``

```
sudo vim /boot/config.txt

```
And add this line to it

```arm_64bit=1```

Then run ``sudo rpi-update``.


Make sure the hostname of the pi is "node" followed by a number of your choice. For simplicity, we have node1, node2, node3 and node0 (master).

Edit /boot/cmdline.txt in your editor of choice (mine being vim):

```
sudo vim /boot/cmdline/txt
```


Add ``cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory`` to the END of the line. Do NOT create a new line!

At this stage you must restart the pis for the changes to take affect. ``sudo reboot``


On the laptop 

```
curl -sfL https://get.k3s.io | sh -

sudo k3s kubectl get node

```
The second command should show that there is one "node", which is "node0". Kubernetes has conveniently given us a token. We must use this to connect our slave nodes to the master node.


```sudo cat /var/lib/rancher/k3s/server/node-token```

For each slave node (adjust the environmental variables as needed):

```

 curl -sfL https://get.k3s.io | K3S_URL=https://node1:6443 K3S_TOKEN=token_from_earlier sh -

 ```
Now hold up for a couple of minutes for it to register the slave node.


Hop back over to the laptop:

```

sudo k3s kubectl get node

```

If the registration is crapping out on you run ``sudo service k3s-agent restart`` - nothing the old restart trick can't fix.

This should show that the slave nodes are registered to the master node:


<h3>Getting a GUI going!</h3>


Now what we want is to have like a GUI to click buttons and have stuff done for us in order to make it easier for everyone:


On the laptop run the following

```

sudo kubectl apply -n portainer -f https://raw.githubusercontent.com/Krish-sysadmin/k8s/master/deploy/manifests/portainer/portainer.yaml

```


Now the nice web GUI will magically appear in a couple of seconds:

Go to http://node1:20777


Click on Kubernetes (the "container orchestration system", in simple words what allows us to well make the cluster work) and click save:

We'll see the cluster on the home page (woohoo) with "4 nodes" writen besides it.


<h3>Does it even work?</h3>

- Click on the cluster and "Applications + Add". We'll spin up multiple (data won't be shared across nodes/containers so we'll probs have to setup a central volume) Global instance of whatever app we want to test whether redundancy works (like a website for instance or online VSCode thing - we can decide with Marcus then). Select reasonabe CPU/memory amounts and container port. Click start.

Hope it is working so far. Now with this the possibilities for us are endless. YOU pick what you want to do!
