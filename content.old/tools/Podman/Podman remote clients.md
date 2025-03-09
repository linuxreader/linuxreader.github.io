Don't use this because you are setting up a daemon. Just use ssh instead. 

https://www.redhat.com/en/blog/podman-clients-macos-windows

![](../../images/Pasted%20image%2020241222034820.png)
# Create connection between client and server

Enable Podman service on the server:
```bash
systemctl --user enable podman.socket
```

Enable lingering for your user:
```bash
sudo loginctl enable-linger $USER
```

Make sure qemu-img is installed:
`sudo dnf install qemu-img`
`sudo dnf -y install qemu-img.x86_64`

Install gvproxy
`sudo dnf install -y podman-gvproxy`


Initialize and start a Podman machine: (this may take some time)
```bash
podman machine init
podman machine start
```

Verify the installation:
```bash
podman info
```

Verify the socket is listening: 
```bash
podman --remote info
```