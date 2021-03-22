# Aptly

When trying to update several computers in local network (without internet connection) aptly is a convinient solution.

You can either create a mirror of remote repositories or build your desired repository from deb files.
Aptly snapshot option allows you to transition your package environment to a new version, or rollback to a previous version.

After publishing a snapshot, you can use apt tool in order to install packages.

## Create a key
In your home directory create a GPG2 batch file:
```
cat >~/gpg2_generate_batch_file.txt <<EOF
%echo Generating a default key
Key-Type: RSA
Key-Length: 4096
Name-Real: MyCompanyName
Name-Comment: aptly key no passphrase
Name-Email: info@mycompanyname.com
Expire-Date: 0
%no-protection
# Do a commit here, so that we can later print "done" :-)
%commit
%echo done
EOF
```

Generate a general key:

```
gpg --batch --gen-key ~/gpg2_generate_batch_file.txt
```

This key will allow us to sign and export our repository. First we need to check our key:
``` 
gpg --list-key
```
The key will look as follow:
```
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
/home/robokal/.gnupg/pubring.kbx
------------------------------
pub   rsa4096 2021-03-20 [SCEA]
      152B12FDCCEB6DCD186430F56FBBF490B10AB3CB
uid           [ultimate] MyCompanyName (aptly key no passphrase) <info@mycompanyname.com>
```
At last, we should export the key:
```
gpg --output ~/.aptly/public/key.pub --armor --export 152B12FDCCEB6DCD186430F56FBBF490B10AB3CB
```

## Publish a reposiroty

Create a repository:
```
aptly repo create repo_name
```

Add packages to your repository (for deb files):

```
aptly repo add repo_name ~/Downloads/folder_name
```

Create a snapshot:

```
aptly snapshot create snap_name from repo repo_name
```

Publish the repository, choose the component and the distribution:
```
sudo aptly publish snapshot -component=main -distribution=focal snap_name
```

Finally, serve the published repository:
```
aptly serve
```


## Use the repository

In other machine, you can use the published repository to install the packages.

First, add the key. Insert the IP of the serve machine:
```
wget http://192.168.1.14:8080/key.pub
sudo apt-key add key.pub
```

Check your machine keys:

```
apt-key list
```

Now you can update your system. 
Add to the file /etc/apt/sources.list (according to the desired serve IP, distribution and component):

deb http://192.168.1.14:8080/ubuntu focal main


At last, you can update your machine:
```
sudo apt update
sudo apt install <package_name>
```



