## Hostname

- "-", "_ ", and ". " characters are allowed.
- Up to 253 characters.
- Stored in */etc/hostname*.
- Can be viewed with several different commands, such as `hostname`, `hostnamectl`, `uname`, and `nmcli`, as well as by displaying the content of the */etc/hostname* file. 

View the hostname:
```
hostnamectl --static
```

```
hostname
```

```
uname -n
```

```
cat /etc/hostname
```

### Lab: Change the Hostname

Server1

1. Open */etc/hostname* and change the entry to `server10.example.com`
2. Restart the `systemd-hostnamed` service daemon
```
sudo systemctl restart systemd-hostnamed
```
3. confirm
```
hostname
```

server2

1. Change the hostname with hostnamectl:
```bah
sudo hostnamectl set-hostname server21.example.com
```
2. Log out and back in for the prompt to update

3. Change the hostname using nmcli
```
nmcli general hostname server20.example.com
```