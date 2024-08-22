# Director Install


### register subscription/satellite, set repos, and install/update
```
  sudo subscription-manager release --set=8.4
  sudo subscription-manager repos --disable=*
  sudo subscription-manager repos \
  --enable=rhel-8-for-x86_64-baseos-eus-rpms \
  --enable=rhel-8-for-x86_64-appstream-eus-rpms \
  --enable=rhel-8-for-x86_64-highavailability-eus-rpms \
  --enable=ansible-2.9-for-rhel-8-x86_64-rpms \
  --enable=openstack-16.2-for-rhel-8-x86_64-rpms \
  --enable=fast-datapath-for-rhel-8-x86_64-rpms

...

sudo dnf module reset container-tools

sudo dnf module enable -y container-tools:3.0

sudo dnf update -y

sudo dnf install -y NetworkManager-config-server python3-ipalib \
python3-ipaclient krb5-devel bind-utils git tmux ipmitool

sudo dnf erase -y cloud-init

sudo reboot

sudo dnf install -y python3-tripleoclient

sudo openstack tripleo container image prepare default \
  --local-push-destination \
  --output-env-file containers-prepare-parameter.yaml.default

```



### Deploy undercloud


## Configure settings CPU architecture for Overcloud
sudo dnf install rhosp-director-images rhosp-director-images-ipa-x86_64

mkdir /home/stack/images

cd ~/images
for i in /usr/share/rhosp-director-images/overcloud-full-latest-16.2.tar /usr/share/rhosp-director-images/ironic-python-agent-latest-16.2.tar; do tar -xvf $i; done

openstack overcloud image upload --image-path /home/stack/images/

openstack image list

ls -l /var/lib/ironic/httpboot




### setup







### Confirm Undercloud Serives/Containers are running
sudo podman ps -a --format "{{.Names}} {{.Status}}"
