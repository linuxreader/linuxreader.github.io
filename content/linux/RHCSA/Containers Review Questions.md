## Containers Review Questions 

Q\. Rootless containers are less secure than rootful containers. True or False?
A\. False.

---

Q\. Which of these---isolation, security, high transition time, less overhead---is not a benefit of the container technology?
A\. High transition time is not a container benefit.

---

Q\. What is the name and location of the system-wide file that stores a list of insecure registries?
A\. The file that stores insecure registry list is called *registries.conf* and it lives in the */etc/containers* directory.

---

Q\. What is one thing that you do not get with rootless containers? Select from: security, ability to map privileged ports, ability to pass environment variables, or can be managed via systemd.
A\. The ability to map privileged ports is not directly supported with rootless containers.

---

Q\. What is the significance of a tag in a container image?
A\. A tag is typically used to identify a version of the image.

---

Q\. Which command would you use to inspect a container image sitting in a remote registry?
A\. The `skopeo` command can be used to inspect an image located in a remote registry.

---

Q\. What would you run to remove all locally stored images in one shot?
A\. Execute  `podman rmi -a` to remove all locally stored images.

---

Q\. What is the default tag used with container images?
A\. The default tag is 'latest' that identifies the latest version of an image.

---

Q\. The per-user systemd service configuration file is stored under */etc/systemd/user* directory. True or False?
A\. False. The per-user systemd unit configuration file is stored under the user's home directory at *~/.config/systemd/user*.

---

Q\. Containers can be run on a bare metal server or in a virtual machine. True or False?
A\. True.

---

Q\. Which feature would you use to allow traffic flow for your Apache web server running inside a container?
A\. Map a host port with a container port.

---

Q\. How would you ensure that data stored inside a container will not be deleted automatically when the container is restarted?
A\. Attach a host directory to the container and use it for storing data that requires persistence.

---

Q\. What is the name of the primary tool that is used for container management?
A\. The primary container management tool is called `podman`.

---

Q\. Which option must be included with the `systemctl` command to manage the state of a rootless container?
A\. The `--user` option is required with the `systemctl` command to control the state of a rootless container.

---

Q\. It is mandatory to download an image prior to running a container based off it. True or False?
A\. False. The specified image is automatically downloaded when starting a container.

---

Q\. How would you connect to a container running in the background?
A\. Use the podman attach command to connect to a running container.

---

Q\. By default, any data saved inside a container is lost when the container is restarted. True or False?
A\. True.

---

Q\. What is the typical name of the file that furnishes instructions to build custom images?
A\. *Containerfile* is the typical name; however, you can use any name that you want.

---

Q\. Can a single host directory be attached to multiple containers at the same time?
A\. Yes, a single host directory can be attached to multiple containers simultaneously.

---

Q\. How would you ensure that a rootless container for a specific user is automatically started with the host startup without the need for the
user to log in?
A\. Use the `loginctl` command as that user and enable `lingering`.

---

Q\. What would you run to remove all stopped containers?
A\. Run `podman rm -a` to remove all stopped containers.
