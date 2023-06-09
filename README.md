# Yocto & Rauc Demo

See

* https://github.com/rauc/meta-rauc
* https://rauc.readthedocs.io/en/latest/

**Create directory structure**

and clone this repo into directory `poky/meta-demorauc`.

```
poky
poky/meta-demorauc 
```


**Clone other layers**

```
cd poky
./meta-demorauc/run_1_yocto_bootstrap_git.sh
```

**Start docker container**

```
cd poky
docker run --rm -it -v `pwd`:/workdir crops/poky --workdir=/workdir
```

Build

```
source ./meta-demorauc/run_2_docker_init-build-env.sh
bitbake rauc-bundle-demo
bitbake core-image-minimal-demo
runqemu qemux86-64 slirp nographic core-image-minimal-demo

runqemu slirp nographic tmp/deploy/images/qemux86-64/core-image-minimal-demo-qemux86-64.qemuboot.conf
```

Create disk image

```
wic create mkefidisk -e core-image-minimal-demo
```

Outside of docker
```
sudo dd if=mkefidisk-202305072037-sda.direct of=/dev/sda
sync
```

## Create keys

* See https://rauc.readthedocs.io/en/latest/examples.html#full-system-example

```
openssl req -x509 -newkey rsa:4096 -nodes -keyout demorauc.key.pem -out demorauc.cert.pem -subj "/O=rauc Inc./CN=rauc-demo"
```

## Current issues - A

```
WARNING: rauc-1.9+gitAUTOINC+dab6882390-r0 do_install: Please overwrite example system.conf with a project specific one!
WARNING: rauc-1.9+gitAUTOINC+dab6882390-r0 do_install: Please overwrite example ca.cert.pem with a project specific one, or set the RAUC_KEYRING_FILE variable with your file!
```

**dbus points to wrong socket**

* https://www.phytec.de/cdocuments/?doc=A4AGG&cHash=f5f76292d778ec54a7602e840cf7a01b
* https://github.com/rauc/rauc/issues/722


```
rauc status
grep -R /run/dbus/system_bus_socket /usr/share/
rm /var/run/dbus/system_bus_socket
ln -s /run/dbus/system_bus_socket /var/run/dbus/system_bus_socket
```

Workaround

```
root@qemux86-64:/var# DBUS_SESSION_BUS_ADDRESS=unix:path=/run/dbus/system_bus_socket
root@qemux86-64:/var# rauc status
```

