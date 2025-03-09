


### Controlling access

creates accounts for new users, removes the accounts of inactive users, and handles all the account-related issues that come up in between

process of actually adding and removing accounts is typically automated by a configuration management system or centralized directory service


### Adding hardware

### Automating tasks

 increases your efficiency, reduces the likelihood of errors caused by humans, and improves your ability to respond rapidly to changing equirements

### Overseeing backups

Backups must be executed on a regular schedule and restores must be tested periodically to ensure that they are functioning correctly.

### Installing and upgrading software

As patches and security updates are released, they must be tested, reviewed, and incorporated into the local environment without endangering the stability of production systems

software delivery” refers to the process of releasing updated versions of software

Continuous delivery” takes this process to the next level by automatically releasing software to users at a regular cadence as it is developed

### Monitoring

detecting problems and fixing them before public failures occur.

monitoring tasks include ensuring that web services respond quickly and correctly, collecting and analyzing log files, and keeping tabs on the availability of server resources such as disk space

### Troubleshooting


### Maintaining local documentation

Thorough and accurate documentation is a blessing for team members who would otherwise need to reverse-engineer a system to resolve problems in the middle of the night


## Vigilantly monitoring security

must implement a security policy and set up procedures to prevent systems from being breached

might include only a few basic checks for unauthorized access, or it might involve an elaborate network of traps and auditing programs, depending on the context

### Tuning performance
tailor systems for optimal performance in accord with the needs of users, the available infrastructure, and the services the systems provide

 When a server is performing poorly, it is the administrator’s job to investigate its operation and identify areas that need improvement.

### Developing site policies

most sites need policies that govern the acceptable use of computer systems, the management and retention of data, the privacy and security of networks and systems, and other areas of regulatory interest

develop sensible policies that meet the letter and

intent of the law and yet still promote progress and productivity

### Working with vendors

### Fire fighting

your response to these issues affects your perceived value as an administrator far more than does any actual technical skill you might possess a single well-handled trouble ticket scores more brownie points than five hours of midnight debugging

### Vim
Mastering vim is perhaps the single best productivity enhancement available to administrators
Use the vimtutor command for an excellent, interactive introduction.

### Programming
Capable administrators are usually polyglot programmers who don’t mind picking up a new language when the need arises

we recommend Bash (aka bash, aka sh), Ruby, or Python

We also suggest that you learn expect, which is not a programming language so much as a front end for driving interactive programs. It’s an efficient glue technology that can replace some complex scripting and is easy to learn.

### Distributions

A Linux distribution comprises the Linux kernel, which is the core of the operating system, and packages that make up all the commands you can run on the system

All distributions share the same kernel lineage, but the format, type, and number of packages
differ quite a bit

distributions derived from the Debian and Red Hat lineages will predominate in production environments in the years ahead

Virtually all leading distributions are cluttered with scores of unused software packages; security vulnerabilities and administrative anguish often come along for the ride






> By adopting a distribution, you are making an investment in a
> particular vendor’s way of doing things. Instead of looking only at
> the features of the installed software, it’s wise to consider how your
> organization and that vendor are going to work with each other. Some
> important questions to ask are:






> 1.  Is this distribution going to be around in five years?
> 2.  Is this distribution going to stay on top of the latest security
>     patches?






> Does this distribution have an active community and sufficient
> documentation






> If I have problems, will the vendor talk to me, and how much will that
> cost






> A comprehensive list of distributions, including many non-English
> distributions, can be found at lwn.net/Distributions or
> distrowatch.com






> Debian GNU/Linux, Ubuntu Linux, Red






> Hat Enterprise Linux (and its dopplegänger CentOS), and FreeBSD






> Debian defines three releases that are maintained simultaneously:
> stable, targeting production servers; unstable, with current packages
> that may






> have bugs and security vulnerabilities; and testing, which is
> somewhere in between.






> Ubuntu version numbers derive from the






> year and month of release, so version 16.10 is from October, 2016.
> Each release also has an alliterative code name such as Vivid Vervet
> or Wily Werewolf.






> Two versions of Ubuntu are released annually: one in April and one in
> October. The April releases






> in even-numbered years are long-term support (LTS) editions that
> promise five years of maintenance updates. These are the releases
> recommended for production use






> RHEL is open source but requires a license. If you’re not willing to
> pay for the license, you’re not going to be running Red Hat.






> Fedora, a community-based distribution that serves as an incubator for
> bleeding-edge software not considered stable enough for RHEL






> Fedora is used as the initial test bed for software and configurations
> that later find their way to RHEL






> CentOS is an excellent choice for sites that want to deploy a
> production-oriented distribution without paying tithes to Red Hat. A
> hybrid approach is also feasible: front-line servers can run Red Hat
> Enterprise






> Linux and avail themselves of Red Hat’s excellent support, even as
> nonproduction systems run CentOS






> This arrangement covers the important bases in terms of risk and
> support while also minimizing cost and administrative complexity






> CentOS aspires to full binary and bug-for-bug compatibility with Red
> Hat Enterprise Linux






> Apple’s macOS has a BSD heritage






> FreeBSD






> commands a 70% market share among BSD variants






> Unlike Linux, FreeBSD is a complete operating system, not just a
> kernel






> We use $ to denote the shell prompt for a normal, unprivileged user,
> and # for the root user






> 1.  A star (\*) matches zero or more characters.
> 2.  A question mark (?) matches one character.
> 3.  A tilde or “twiddle” (~) means the home directory of the current
>     user






> ~user means the home directory of user.






> one “megabyte” of memory is really 220 or 1,048,576 bytes






> RAM is always denominated in powers of 2, but network bandwidth






> is always a power of 10. Storage space is usually quoted in
> power-of-10 units, but block and page sizes are in fact powers of 2






> Unit decoding examples






> see wiki.ubuntu.com/UnitsPolicy for some additional details






> Man pages and other on-line documentatio






> consult man pages as an authoritative resource because they are
> accessible from the command line






> details on a program’s options, and show helpful examples and related
> commands






> concise descriptions of individual commands, drivers, file formats, or
> library routines






> FreeBSD and Linux divide the man pages into sections.






> Sections of the man pages






> be aware of the section definitions when a topic with the same name
> appears in multiple sections






> man title formats a specific manual page and sends it to your terminal
> through more, less, or whatever program is specified in your PAGER
> environment variable






> title is usually a command, device, filename, or name of a library
> routine






> sections of the manual are searched in roughly numeric order, although
> sections that describe commands (sections 1 and 8) are usually
> searched first.






> man section title gets you a man page from a particular section






> man sync gets you the man page for the sync command, and man 2 sync
> gets you the man page for the sync system call






> man -k keyword or apropos keyword prints a list of man pages that have
> keyword in their one-line synopses






> The keywords database can become outdated. If you add additional man
> pages to your system






> you may need to rebuild this file with makewhatis (Red Hat and
> FreeBSD) or mandb (Ubuntu).






> nroff input for man pages (i.e., the man page source code) is stored
> in directories under /usr/share/man and compressed with gzip to save
> space






> man maintains a cache of formatted pages in /var/cache/man or
> /usr/share/man if the appropriate directories are writable; however,
> this is a security risk. Most systems preformat the man pages once at
> installation time (see catman) or not at all.






> The man command can search several man page repositories to find the
> manual pages you request. On Linux systems, you can find out the
> current default search path with the manpath command. This path (from
> Ubuntu) is typical






> If necessary, you can set your MANPATH environment variable to
> override the default path






> Some systems let you set a custom system-wide default search path for
> man pages, which can be useful if you need to maintain a parallel tree
> of man pages such as those generated by OpenPKG






> Where to find OS






> Pro Git from git-scm.com/book make the hunt worthwhile






> Many pieces of software have both a man page and a long-form article






> Most software projects have user and developer mailing lists and IRC
> channels






> if you have questions






> 
> Request for Comments documents describe the protocols and procedures
> used on the Internet






> The phrase “reference implementation” applied to software






> usually translates to “implemented by a trusted source according to
> the RFC specification






> When stuck, search the web.






> Resources for keeping up to date






> <div>
> <div>
> <div></div>
> </div>
> </div>
> 
> Social media are also useful. Twitter and reddit in particular have
> strong, engaged communities with a lot to offer, though the
> signal-to-noise ratio can sometimes be quite bad. On reddit, join the
> sysadmin, linux, linuxadmin, and netsec subreddits.






> Task-specific forums and reference sites






> Stack Overflow and Server Fault






> It’s worth creating an account and joining this large community






> Conferences relevant to system administrators






> Meetups (meetup.com) are another way to network and engage with
> like-minded people. Most urban areas in the United States and around
> the world have a Linux user group or DevOps meetup that sponsors
> speakers, discussions, and hack days.






> When adding software, don your security hat and remember that
> additional software creates additional attack surface






> Most software is developed by independent groups that release the
> software in the form of source code. Package repositories then pick up
> the source code, compile it appropriately for the conventions in use
> on the systems they serve, and package the resulting






> binaries. It’s usually easier to install a system-specific binary
> package than to fetch and compile the original source code. However,
> packagers are sometimes a release or two behind the current version.






> two systems use the same package format doesn’t necessarily mean that
> packages for the two systems are interchangeable






> Red Hat and SUSE both use RPM






> When the packaged format is insufficient, administrators must install
> software the old-fashioned way: by downloading a tar archive of the
> source code and manually configuring, compiling, and installing it.
> Depending on the software and the operating system, this process can
> range from trivial to nightmarish






> use the shell’s which command to find out if a relevant binary is
> already






> reveals that the GNU C compiler has already been installed on this
> machine






> If which can’t find the command you’re looking for, try whereis; it
> searches a broader range of system directories and is independent of
> your shell’s search path






> locate command, which consults a precompiled index






> the filesystem to locate filenames that match a particular pattern.






> FreeBSD includes locate as part of the base system. In Linux, the
> current implementation of locate is in the mlocate package






> On Red Hat






> and CentOS, install the mlocate package with yum; see this page.






> locate can find any type of file; it is not specific to commands or
> packages. For example, if you weren’t sure where to find the signal.h
> include file, you could try






> locate’s database is updated periodically by the updatedb command (in
> FreeBSD






> locate.updatedb), which runs periodically out of cron.






> Therefore, the results of a locate don’t always reflect recent changes
> to the filesystem






> you can also use your system’s packaging utilities to check directly
> for the package’s presence



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#45|1713)



> checks for the presence (and installed version) of the Python
> interpreter



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#45|1828)



> find out which package a particular file belongs to:
> 



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#46|557)



> installation of the tcpdump command on each of our example systems.
> tcpdump is a packet capture tool that lets you view the raw packets
> being sent to and from the system on the network.
> 
> <div>
> <div></div>
> </div>
> 
> <div>
> <div></div>
> </div>
> 
> Debian and Ubuntu use APT, the Debian Advanced Package Tool:



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#46|931)



> The Red Hat and CentOS version is



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#46|1016)



> The package manager for FreeBSD is pkg.



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#47|75)



> build a version of tcpdump from the source code.



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#47|150)



> identify the code.



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#47|169)



> Software maintainers sometimes keep an index of releases on the
> project’s web site that are downloadable as tarballs. For open source
> projects, you’re most likely to find the code in a Git repository.



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#47|373)



> The tcpdump source is kept on GitHub. Clone the repository in the /tmp
> directory, create a branch of the tagged version you want to build,
> then unpack, configure, build, and install it:



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#47|590)



> configure/make/make install sequence is common to most software
> written in C and works on all UNIX and Linux systems.



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#47|736)



> check the package’s INSTALL or README file for specifics



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#47|798)



> must have the development environment and any package-specific
> prerequisites installed. (In the case of tcpdump, libpcap and its
> libraries are prerequisites.)



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#47|960)



> You’ll often need to tweak the build configuration, so use ./configure
> --help to see the options available for each particular package.
> Another useful configure option is --prefix=directory, which lets you
> compile the software for installation somewhere other than /usr/local,
> which is usually the default.



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#48|34)



> Cross-platform software bundles increasingly offer an expedited
> installation process that’s driven by a shell script you download from
> the web with curl, fetch, or wget.



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#48|204)



> These are all simple HTTP clients that download the contents of a URL
> to a local file or, optionally, print the contents to their standard
> output. For example, to set up a machine as a Salt client, you can run
> the following commands:
> 
> <div>
> <div></div>
> </div>



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#48|465)



> The bootstrap script investigates the local environment, then
> downloads, installs, and configures an appropriate version of the
> software. This type of installation is particularly common in cases
> where the process itself is somewhat complex, but the vendor is highly
> motivated to make things easy for users. (Another good example is RVM;
> see this page.)



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#48|908)



> This installation method



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#48|1030)



> leaves no proper record of the installation for future reference. If
> your operating system offers a packagized version of the software,
> it’s usually preferable to install the package instead of running a
> web installer.



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#48|1249)



> Packages are easy to track, upgrade, and remove. (On the other hand,
> most OS-level packages are out of date. You probably won’t end up with
> the most current version of the software.)



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#48|1511)



> Be very suspicious if the URL of the boot script is not secure (that
> is, it does not start with https:). Unsecured HTTP is trivial to
> hijack, and installation URLs are of particular interest to hackers
> because they know you’re likely to run, as root, whatever code comes
> back. By contrast, HTTPS validates the identity of the server through
> a cryptographic chain of trust. Not foolproof, but reliable enough.



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#48|1923)



> A few vendors publicize an HTTP installation URL that automatically
> redirects to an HTTPS version. This is dumb and is in fact no more
> secure than straight-up HTTP. There’s nothing to prevent the initial
> HTTP exchange from being intercepted, so you might never reach the
> vendor’s redirect



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#48|2213)



> However, the existence of such redirects does mean it’s worth trying
> your own substitution of https for http in insecure URLs. More often
> than not, it works just fine.



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#48|2618)



> he root shell still runs even if curl outputs a partial script and
> then fails—say, because of a transient network glitch. The end result
> is unpredictable and potentially not good.



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#48|2947)



> piping the output of curl to a shell has entered the collective
> sysadmin unconscious as a prototypical rookie blunder, so if you must
> do it, at least keep it on the sly.



          1.10 Ways to find and install software
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#48|3138)



> ust save the script to a temporary file, then run the script in a
> separate step after the download successfully completes.



          1.11 Where to host
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#49|326)



> The most practical choice, and our recommendation for new projects, is
> a public cloud provider. These facilities offer numerous advantages
> over data centers:
> 
> * No capital expenses and low initial operating costs
> * No need to install, secure, and manage hardware
> * On-demand adjustment of storage, bandwidth, and compute capacity
> * Ready-made solutions for common ancillary needs such as databases,
>   load balancers, queues, monitoring, and more
> * Cheaper and simpler implementation of highly available/redundant
>   systems



          1.11 Where to host
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#49|1106)



> Our preferred cloud platform is the leader in the space: Amazon Web
> Services (AWS)



          1.11 Where to host
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#49|1246)



> AWS is ten times the size of all competitors combined



          1.11 Where to host
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#49|1301)



> AWS innovates rapidly and offers a much broader array of services than
> does any other provider.



          1.11 Where to host
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#49|1510)



> free service tier to cut your teeth on, including a year’s use of a
> low powered cloud server.



          1.11 Where to host
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#49|1607)



> Google Cloud Platform (GCP



          1.11 Where to host
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#49|1805)



> reputation for dropping support for popular offerings.



          1.11 Where to host
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#49|1873)



> customer-friendly pricing terms



          1.11 Where to host
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#49|1959)



> DigitalOcean



          1.11 Where to host
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#49|1977)



> simpler service with a stated goal of high performance.



          1.11 Where to host
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#49|2037)



> target market is developers, whom it woos with a clean API, low
> pricing, and extremely fast boot times



          1.11 Where to host
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#49|2159)



> strong proponent of open source software, and their tutorials and
> guides for popular Internet technologies are some of the best
> available.



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#50|347)



> Your goal as a system administrator, or as a professional working in
> any of these related areas, is to achieve the objectives of the
> organization. Avoid letting politics or hierarchy interfere with
> progress. The best administrators solve problems and share information
> freely with others.



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#51|78)



> DevOps



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#51|163)



> aims to improve the efficiency of building and delivering software,
> especially at large sites that have many interrelated services and
> teams. Organizations with a DevOps practice promote integration among
> engineering teams and may draw little or no distinction between
> development and operations. Experts who work in this area seek out
> inefficient processes and replace them with small shell scripts or
> large and unwieldy Chef repositories.



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#52|32)



> Site reliability engineers value uptime and correctness above all else



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#52|104)



> Monitoring networks, deploying production software, taking pager duty,
> planning future expansion, and debugging outages



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#52|282)



> Single points of failure are site reliability engineers’ nemeses.



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#53|35)



> Security operations engineers



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#53|65)



> focus on the practical, day-to-day side of an information security
> program



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#53|153)



> install and operate tools that search for vulnerabilities and monitor
> for attacks on the network. They also participate in attack
> simulations to gauge the effectiveness of their prevention and
> detection techniques.



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#54|28)



> Network administrators design, install, configure, and operate
> networks. Sites that operate data centers are most likely to employ
> network administrators; that’s because these facilities have a variety
> of physical switches, routers, firewalls, and other devices that need
> management. Cloud platforms also offer a variety of networking
> options, but these usually don’t require a dedicated administrator
> because most of the work is handled by the provider.



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#55|2)



> Database administrators



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#55|83)



> experts at installing and managing database software.



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#55|142)



> manage database schemas, perform installations and upgrades, configure
> clustering, tune settings for optimal performance, and help users
> formulate efficient queries.



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#55|317)



> usually wizards with one or more query languages and have experience
> with both relational and nonrelational (NoSQL) databases.



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#56|47)



> NOC engineers monitor the real-time health of large sites and track
> incidents and outages.



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#56|143)



> troubleshoot tickets from users, perform routine upgrades, and
> coordinate actions among other teams. They can most often be found
> watching a wall of monitors that show graphs and measurements.



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#57|2)



> Data center technicians



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#57|53)



> work with hardware. They receive new equipment, track equipment
> inventory and life cycles, install servers in racks, run cabling,
> maintain power and air conditioning, and handle the daily operations
> of a data center



          1.12 Specialization and adjacent disciplines
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#58|16)



> Systems architects have deep expertise in more than one area. They use
> their experience to design distributed systems. Their job descriptions
> may include defining security zones and segmentation, eliminating
> single points of failure, planning for future growth, ensuring
> connectivity among multiple networks and third parties, and other
> site-wide decision making. Good architects are technically proficient
> and generally prefer to implement and test their own designs.



          1.13 Recommended reading
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#59|30)



> Abbott, Martin L., and Michael T. Fisher. The Art of Scalability:
> Scalable Web Architecture, Processes, and Organizations for the Modern
> Enterprise (2nd Edition). Addison-Wesley Professional, 2015.
> 
> Gancarz, Mike. Linux and the Unix Philosophy. Boston: Digital Press,
> 2003.
> 
> Limoncelli, Thomas A., and Peter Salus. The Complete April Fools’ Day
> RFCs. Peer-to-Peer Communications LLC. 2007. Engineering humor. You
> can read this collection on-line for free at rfc-humor.com.
> 
> Raymond, Eric S. The Cathedral &amp; The Bazaar: Musings on Linux and
> Open Source by an Accidental Revolutionary. Sebastopol, CA: O’Reilly
> Media, 2001.
> 
> Salus, Peter H. The Daemon, the GNU &amp; the Penguin: How Free and
> Open Software is Changing the World. Reed Media Services, 2008. This
> fascinating history of the open source movement by UNIX’s best-known
> historian is also available at groklaw.com under the Creative Commons
> license. The URL for the book itself is quite long; look for a current
> link at groklaw.com or try this compressed equivalent:
> tinyurl.com/d6u7j.
> 
> Siever, Ellen, Stephen Figgins, Robert Love, and Arnold Robbins. Linux
> in a Nutshell (6th Edition). Sebastopol, CA: O’Reilly Media, 2009.



          1.13 Recommended reading
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#60|2)



> 
> Kim, Gene, Kevin Behr, and George Spafford. The Phoenix Project: A
> Novel about IT, DevOps, and Helping Your Business Win. Portland, OR:
> IT Revolution Press, 2014. A guide to the philosophy and mindset
> needed to run a modern IT organization, written as a narrative. An
> instant classic.
> 
> Kim, Gene, Jez Humble, Patrick Debois, and John Willis. The DevOps
> Handbook: How to Create World-Class Agility, Reliability, and Security
> in Technology Organizations. Portland, OR: IT Revolution Press, 2016.
> 
> Limoncelli, Thomas A., Christina J. Hogan, and Strata R. Chalup. The
> Practice of System and Network Administration (2nd Edition). Reading,
> MA: Addison-Wesley, 2008. This is a good book with particularly strong
> coverage of the policy and procedural aspects of system
> administration. The authors maintain a system administration blog at
> everythingsysadmin.com.
> 
> Limoncelli, Thomas A., Christina J. Hogan, and Strata R. Chalup. The
> Practice of Cloud System Administration. Reading, MA: Addison-Wesley,
> 2014. From the same authors as the previous title, now with a focus on
> distributed systems and cloud computing.



          1.13 Recommended reading
        ](https://reader.bookfusion.com/books/4031036-unix-and-linux-system-administration-handbook-fifth-edition?type=epub_reflowable#61|2)



> 
> Blum, Richard, and Christine Bresnahan. Linux Command Line and Shell
> Scripting Bible (3rd Edition). Wiley, 2015.
> 
> Dougherty, Dale, and Arnold Robins. Sed &amp; Awk (2nd Edition).
> Sebastopol, CA: O’Reilly Media, 1997. Classic O’Reilly book on the
> powerful, indispensable text processors sed and awk.
> 
> Kim, Peter. The Hacker Playbook 2: Practical Guide To Penetration
> Testing. CreateSpace Independent Publishing Platform, 2015.
> 
> Neil, Drew. Practical Vim: Edit Text at the Speed of Thought.
> Pragmatic Bookshelf, 2012.
> 
> Shotts, William E. The Linux Command Line: A Complete Introduction.
> San Francisco, CA: No Starch Press, 2012.
> 
> Sweigart, Al. Automate the Boring Stuff with Python: Practical
> Programming for Total Beginners. San Francisco, CA: No Starch Press,
> 2015.





## AutoFS

- Automatically mount and unmount on clients during runtime and system reboots.
- Triggers mount or unmount action based on mount point activity.
- Client-side service
- Mount an NFS share on demand
- Entry placed in AutoFS config files.
- Automatically mounts share upon detecting activity in it's mount point. (touch, ls, cd)
- unmounts share if the share hasn't been accessed for a predefined period of time. 
- Mounts managed with autofs should not be mounted manually via /etc/fstab to avoid inconsistencies.
- Saves Kernel from having to maintain unused NFS shares. (Improved performance!)
- NFS shares are defined in config files called maps (/etc/ or /etc/auto.master.d/)
- Does not use /etc/fstab.
- Does not require root to mount a share (fstab does).
- Prevents client from hanging if share is down.
- Share is unmounted if not accessed for 5 minutes (default)
- Supports wildcard characters or environment variables.
- Automount daemond 
  - in the userland mounts configured shares automatically upon access.
  - invoked at system boot.
  - Reads AutoFS master map and create inititial mount point entries. (not mounting yet)
  - Does not mount shares until user activity is detected.
  - Unmounts after set timeframe of inactivity.
- Use the mount command on a share to verify the path of the AutoFS map, file system type, and options used during mount.

/etc/autofs.conf/ preset Directives:
master_map_name=auto.master
timeout = 300
negative_timeout = 60
mount_nfs_default_protocol = 4
logging = none



Additional directives:
**master_map_name**
\- Name of the master map. Default is /etc/auto.master
**timeout**
\- time in second to unmount a share.
**negative_timeout**
\- Timeout (in seconds) value for failed mount attempts. (1 minute default)
**mount_nfs_default_protocol**
\- Sets the NFS version used to mount shares.
logging
\- Logging level (none, verbose, debug)
\- Default is none (disabled)

- Normally left to their default values.

### AutoFS Maps

- Where AutoFS finds the shares to mount and their locations.
- Also tells Autofs what options to use. Map Types
- master
- direct
- indirect

**Master Map**

Define entries for indirect and direct maps. 

- /etc/auto.master is default
- Default is defined in /etc/autofs.conf with master_map_name directive.
- May be used to define entries for indirect and direct maps. 
  - But it is recommended to store user-defined maps in /etc/auto.master.d/ 
    - AutoFS service parses this at startup.
- You can append an option to auto.master but it will apply globally to all subentries in the specified map file.

Map entry format examples:
  ----------------------- -------------------------------- -----------------------
  /-                      /etc/auto.master.d/auto.direct   \# Line 1

  /misc                   /etc/auto.misc                   \# Line 2
### Direct Map

```
/- /etc/auto.master.d/auto.direct <-- defines direct map and points to auto.direct for details
```

Mount shares on unrelated mount points

- Always visible to users
- Can exist with an indirect share under one parent directory
- Accessing a directory containing many direct mount points mounts all shares.
- Each direct map entry places a separate share entry to /etc/mtab 
  - /etc/mtab maintains a list of all mounted file systems whether they are local or remote.
  - Updated whenever a local file system, removable file system, or aq network share is mounted or unmounted.

### Indirect Map

```
/misc /etc/auto.misc <-- indirect map and points to auto.misc for details
```

Automount removable filesystems

- Mount point /misc precedes mount point entries in /etc/auto.miscq
- Used to automount removable file systems (CD, DVD, USB disks, etc.)
- Custom indirect map files should be located in /etc/auto.master.d/
- Preferred over direct mount for mounting all shares under one common parent directory.
- Become visible only after they have been accessed.
- Local and indirect mounted shares cannot coexist under the same parent directory.
- One entry in /etc/mtab gets added for each indirect map.
- Accessing a directory containing many indirect mount points shows only the shares that are already mounted.
- Usually better to use indirect map for automounting NFS shares.


### Lab: Access NFS Share Using Direct Map (server10)

1. Install Autofs

```
sudo dnf install -y autofs
```

1. Create mount point /autodir using mkdir

```
sudo mkdir /autodir
```

1. Add an entry to /etc/auto.master to point the AutoFS service to the auto.dir file for more information:

```
/- /etc/auto.master.d/auto.dir
```

1. Create /etc/auto.master.d/auto.dir and add the mount point, NFS server, and share info:

```
/autodir server20:/common
```

1. Start AutoFS service and enable it at startup:

```
sudo systemctl enable --now autofs
```

1. Make sure AUtoFS service is running. Use -l and --no-pager options to show full details without piping the output to a pager program (pg)

```
sudo systemctl status autofs -l --no-pager
```

1. Run ls on the mount point then verify the share is automounted and accessible with mount.

```
ls /autodir
mount | grep autodir
```

1. Wait 5 minutes and run the mount command again to see it has disappeared.

```
mount | grep autodir
```


### Exercise 16-4: Access NFS Share Using Indirect Map

- configure an indirect map to automount the NFS share /common that is available from server20. 
- install the relevant software and set up AutoFS maps to support the automatic mounting.
- observe that the specified mount point "autoindir" is created automatically under /misc.

Note that /common is already mounted on the /local mount point via the fstab file and it is also configured via a direct map for automounting on /autodir. There should occur no conflict in configuration or functionality among the three.

1\. Install the autofs software package if it is not already there:

2\. Confirm the entry for the indirect map /misc in the /etc/auto.master
file exists:
```bash
[root@server30 common]# grep ^/misc /etc/auto.master
/misc	/etc/auto.misc
```

3\. Edit the /etc/auto.misc file and add the mount point, NFS server, and share information to it:
```bash
autoindir server30:/common
```


4\. Start the AutoFS service now and set it to autostart at system reboots:
```bash
[root@server40 /]# systemctl enable --now autofs
```

5\. Verify the operational status of the AutoFS service. Use the -l and \--no-pager options to show full details without piping the output to a pager program (the pg command in this case):
```bash
[root@server40 /]# systemctl status autofs -l --no-pager
```
\
6\. Run the ls command on the mount point /misc/autoindir and then grep for both auto.misc and autoindir on the mount command output to verify that the share is automounted and accessible:
```bash
[root@server40 /]# ls /misc/autoindir
test.text
```

```bash
[root@server40 /]# mount | egrep 'auto.misc|autoindir'
/etc/auto.misc on /misc type autofs (rw,relatime,fd=7,pgrp=3321,timeout=300,minproto=5,maxproto=5,indirect,pipe_ino=31779)
server30:/common on /misc/autoindir type nfs4 (rw,relatime,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.0.40,local_lock=none,addr=192.168.0.30)
```

- /misc/autoindir has been auto generated.
- You can use the ubrella mount point /misc to mount additional auto-generated mount points. 

### Automounting User Home Directories \

AutoFS allows us to automount user home directories by exploiting two special characters in indirect maps.

**asterisk (\*)**
- replaces the references to specific mount points and the 

**ampersand (&)** 
- Substitutes the references to NFS servers and shared subdirectories.

- With user home directories located under /home, on one or more NFS servers, the AutoFS service will connect with all of them simultaneously when a user attempts to log on to a client. 

- The service will mount only that specific user's home directory rather than the entire /home. 

- The indirect map entry for this type of substitution is defined in an indirect map, such as /etc/auto.master.d/auto.home.

`* -rw &:/home/&`

- With this entry in place, there is no need to update any AutoFS configuration files if additional NFS servers with /home shared are added or removed. 

- If user home directories are added or deleted, there will be no impact on the functionality of AutoFS. 

- If there is only one NFS server sharing the home directories, you can simply specify its name in lieu of the first & symbol in the above entry.

### Exercise 16-5: Automount User Home Directories Using Indirect Map


There are two portions for this exercise. The first portion should be done on server20 (NFS server) and the second portion on server10 (NFS client) as user1 with sudo where required.

first portion
- create a user account called user30 with UID 3000.
- add the /home directory to the list of NFS shares so that it becomes available for remote mount.

second portion
- create a user account called user30 with UID 3000, base directory /nfshome, and no home directory. 
- create an umbrella mount point called /nfshome for mounting the user home directory from the NFS server. 
- install the relevant software and establish an indirect map to automount the remote home directory of user30 under /nfshome. 
- observe that the home directory is automounted under /nfshome when you sign in as user30.

On NFS server server20:

1\. Create a user account called user30 with UID 3000 (-u) and assign
password "password1":

```bash
[root@server30 common]# useradd -u 3000 user30
[root@server30 common]# echo password1 | sudo passwd --stdin user30
Changing password for user user30.
passwd: all authentication tokens updated successfully.
```

2\. Edit the /etc/exports file and add an entry for /home (do not modify or remove the previous entry):
`/home server40(rw)`

3\. Export all the shares listed in the /etc/exports file:
```bash
[root@server30 common]# sudo exportfs -avr
exporting server40.example.com:/home
exporting server40.example.com:/common
```

On NFS client server10:

1\. Install the autofs software package if it is not already there:
`dnf install autofs`

2\. Create a user account called user30 with UID 3000 (-u), base home directory location /nfshome (-b), no home directory (-M), and password "password1":
```bash
[root@server40 misc]# sudo useradd -u 3000 -b /nfshome -M user30
[root@server40 misc]# echo password1 | sudo passwd --stdin user30
```

This is to ensure that the UID for the user is consistent on the server and the client to avoid access issues.

3\. Create the umbrella mount point /nfshome to automount the user's home directory:
```bash
sudo mkdir /nfshome
```

4\. Edit the /etc/auto.master file and add the mount point and indirect map location to it:
`/nfshome /etc/auto.master.d/auto.home`

5\. Create the /etc/auto.master.d/auto.home file and add the following information to it:
`* -rw server30:/home/&`

For multiple user setup, you can replace "user30" with the & character, but ensure that those users exist on both the server and the client with consistent UIDs.

6\. Start the AutoFS service now and set it to autostart at system reboots. This step is not required if AutoFS is already running and enabled.
`systemctl enable --now autofs`

7\. Verify the operational status of the AutoFS service. Use the -l and \--no-pager options to show full details without piping the output to a pager program (the pg command):
`systemctl status autofs -l --no-pager`

8\. Log in as user30 and run the pwd, ls, and df commands for verification:
```bash
[root@server40 nfshome]# su - user30
[user30@server40 ~]$ ls
user30.txt
[user30@server40 ~]$ pwd
/nfshome/user30
[user30@server40 ~]$ df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               4.0M     0  4.0M   0% /dev
tmpfs                  888M     0  888M   0% /dev/shm
tmpfs                  356M  5.1M  351M   2% /run
/dev/mapper/rhel-root   17G  2.2G   15G  13% /
/dev/sda1              960M  344M  617M  36% /boot
tmpfs                  178M     0  178M   0% /run/user/0
server30:/common        17G  2.2G   15G  13% /local
server30:/home/user30   17G  2.2G   15G  13% /nfshome/user30
```

EXAM TIP: You may need to configure AutoFS for mounting a remote user
home directory.
