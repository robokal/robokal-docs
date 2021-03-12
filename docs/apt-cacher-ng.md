# Apt cacher ng

It is a package to create a local cache of the Debian mirrors. 

## Server
To get started you should create a server by installing the package:

```
apt install apt-cacher-ng
```

change the cache directory in the configuration file:

```
vim /etc/apt-cacher-ng/acng.conf
```

In the line of CacheDir choose your directory path.

## Client

Define the client configuration in order to know about the proxy:

```
echo 'Acquire::http { Proxy "http://<server_ip>:3142"; }' | sudo tee -a /etc/apt/apt.conf.d/proxy
```

## Check your cacher:

In the client machine: 

```sudo apt update
sudo apt install <package_name>
```

you should be able to see the files in the server machine (according to CacheDir in the acng.conf file).

