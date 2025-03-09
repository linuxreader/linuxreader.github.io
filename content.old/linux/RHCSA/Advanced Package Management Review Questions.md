## Advanced Package Management Review Questions

Q. A module profile is a list of recommended packages organized for purpose-built deployments. True or False?
A. True.

---

Q. What would the `dnf group info Base` command do?
A. The command provided will list all packages in the specified package group.


---

Q: What would you run to list all available profiles for the module postgresql?
A. To view *postgresql* module profiles, you would run `dnf module info \--profile postgresql`

---

Q. Which `dnf` subcommand can be used to list all available repositories?
A. The `repolist` subcommand.

---

Q. What is the use of the -y option with the `dnf group install` and `dnf module remove` commands?
A. `dnf` will not prompt for user confirmation if the `-y` option is used with it.

---

Q. What is the main advantage of using `dnf` over `rpm`?
A. `dnf` resolves and installs dependent packages automatically.

---

Q. What would be the command to install *perl* 5.26?
A. It would be `dnf module install perl:5.26`

---

Q. What `dnf` subcommand would we use to check available updates for installed packages?
A. The `check-update` subcommand.

---

Q. What is the concept of module in RHEL 9?
A. A module is a collection of packages, including the dependent packages, that are required to install an application.

---

Q. What must be the extension of a yum repository file?
A. The extension of a yum repository configuration file must be .repo in order to be recognized as a valid repo file.

---

Q. A module stream is a group of packages organized by version. True or False?
A. True.

---

Q. What would the `dnf list zsh` command do?
A. The command provided will display installed and available `zsh` package.

---

Q. What would the `dnf list installed gnome` command do?
A. The command provided will display all installed packages that contain gnome in their names.

---

Q. How many package names can be specified at a time with the `dnf install` command?
A. There is no limit.

---

Q. Name the two default yum repositories that contain all the packages for RHEL 9?
A. The two yum repositories the BaseOS and AppStream.

---

Q. We can update all the packages within a package group using the `groupupdate` subcommand with `dnf`. True or False?
A. True.

---

Q. What would you run to reset a module so that it is neither enabled nor disabled?
A. You would run `dnf module reset` to reset a module.

---

Q. What would the `dnf info zsh` command do?
A. The command provided will display the header information for the *zsh* package.

---
