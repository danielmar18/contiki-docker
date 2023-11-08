I highly recommend going through the entire instruction list once before starting the process, at least for the Contiki & Docker part. The "Contiki Setup" goes through the actual installation and setup process while "Host v. Docker Workflow" explains exactly how to get the best out the cohabitation of Docker and local terminal. 

This instruction set was made for Apple Macbooks using Silicon processors but should in theory be partly useful for other OS's as well, at least the Contiki Setup itself.
### Contiki Setup
1. Install [Docker](https://docs.docker.com/desktop/) & Git
	1. Sign In/Up in Docker Application
2. Create directory, move into it(`cd <dir_name>`), clone Git repo and move into it (`cd contiki-ng`)
	1. git clone [https://github.com/contiki-ng/contiki-ng.git](https://github.com/contiki-ng/contiki-ng.git) --branch release/v4.9
	2. **IMPORTANT**: Move inside the `contiki-ng` directory before next step
3. Add the Contiki absolute path to an environmental variable
	1. `export CNG_PATH=<absolute-path-to-your-contiki-ng>`
	2. Can be found by running `pwd`
4. Create `contiker` alias for running the Docker image
	1. ```bash
	   alias contiker="docker run                                                           \
               --privileged                                                          \
               --sysctl net.ipv6.conf.all.disable_ipv6=0                             \
               --mount type=bind,source=$CNG_PATH,destination=/home/user/contiki-ng  \
               --device=/dev/ttyUSB0                                                 \
               --device=/dev/ttyUSB1                                                 \
               -ti contiker/contiki-ng"
5. Run `contiker bash` - **ONLY FOR FIRST TIME OR TO CREATE NEW CONTAINER**
	1. This command creates a new container each time it's run
	2. Docker will open, install image, create container instance and open SSH into it.
6. If a container has already been created, use 
	   `docker exec -it <Continer Name>  /bin/bash`
	to SSH into it.
	1. The name can be seen in Docker application or using `docker ps`
7. Once steps 5 finishes or 6 is run, a SSH connection will be opened to the container and if the alias and CNG_PATH was correctly set, you should be inside the `contiki-ng` folder.
![[Pasted image 20231108114100.png]]
##### Docker/Contiki Troubleshoot
- Make sure environmental paths are correct and correctly placed!
	- They can be notoriously tricky
	- Commands like `alias` can also localised to a certain folder so be aware of that
		- You can add a permanent `alias` by using [.bashrc/.zshrc files](https://www.cyberciti.biz/faq/create-permanent-bash-alias-linux-unix/)
- Make sure Docker has all relevant access to folders
- If commands wont work inside Docker, use `sudo` or alter the system Privacy&Security
- If `deamon` causes issue, a couple of things might work:
	- Restart Docker application
	- Delete container and run steps 3 to 5 again, make sure in correct directory
	- Delete image and run steps 3 to 5 again, make sure in correct directory
- Classic "Break Glass in Case of Emergency" is to close&quit the terminal and try again
#### Host v. Docker Workflow
The problem for M1/M2 is the GCC and more importantly, the MSP430 compiler and it's relationship with the Vagrant tool and VM images/ISO's. Docker allows user to work in the same directory, at the same time, both on host and in Docker(As long as a given file is not used by both). This allows for the following workflow:
- Open 2 terminals
	- 1 for Docker by using `contiker bash`
	- 1 to access to host-side files in the directory created during initiation
	- Can test by modifying *hello-world* example 
- Write code on host or on Docker
	- Can be in VS Code or whatever IDE's chosen on host machine
	- If preferred, code can also be written inside Docker image in VIM or EMacs(or whatever flavor is preferred)
- Compile for `TARGET=sky` on Docker.
	- To compile for mote, the MSP430 compiler is needed which is hard to acquire on M1/M2/... processors.
	- Even though Docker image is ARM based, it has all tools needed to compile for the mote.
- Open in Cooja on host machine (Will explore in more details in next chapter)
- Compile `.upload` file on Docker
	- Again, MSP430 causes issues on host machine but even without a device, the `ihex` file can be compiled on Docker by using `make TARGET=sky <file_name>.upload` (Essentially just skipping MOTES=... part)
	- Devices can be configured for Docker container to allow direct upload but can be challenging to get right. 
- Upload on host machine
	- `make TARGET=sky motelist` should work on host machine
	- Using the `make TARGET=sky MOTES=/dev/tty* <file_name>.upload` does indeed work on host machine as long as it doesn't have to compile.
- Log-in through host machine
	- Same as uploading, logging onto the mote works through the local machine. This is way easier than configuring the Docker container to allow the USB connection.

This way, Docker takes care of compiling the actual files while every other step can be done locally like any other development experience. 
###### No Vagrant, No Virtual Machine!

### Cooja Setup
1. Install [OpenJDK@17](https://formulae.brew.sh/formula/openjdk@17#default) through Homebrew
	1. **Important**: Has to be version 17 if used with release 4.9 or so it seems
	2. Make sure the path to the package is in the `$PATH` environmental variable
		1. For example `export PATH=${PATH}:/opt/homebrew/opt/openjdk@17/bin` if installed through Homebrew. Installation notes usually have instructions on how to add it to environment.
		2. If still not working, might need to add to `JAVA_HOME` environmental variable, it should point to the OpenJDK location **without** `/bin` so in this case it would be ``export JAVA_HOME=/opt/homebrew/opt/openjdk@17
2. Move into `contiki-ng/tools/cooja` directory
	1. Run command `git submodule update --init` to install necessary code for Cooja to run
		1. **This will take some time to run. Get a cup of coffee, you deserve it!**
3. Once the command has finished, run `./gradlew run`in the same directory, and Cooja should start right up!
	1. If you compiled a `.sky` file in previous step, try and import into a simulation!

Same as with Contiki, with a bit of Homebrew and environmental path magic, we get a much simpler and more enjoyable process. Both Cooja and Mote development in the same directory, without any VM hassle.
###### No Vagrant, No Virtual Machine!
##### Cooja Troubleshooting
In case the Cooja creation causes trouble, here are some possible step to resolve the issue:
- Restart terminal
	- Sometimes, depending on configuration and terminal application, you need to fully close down the terminal and stop all executions in order for it to work
- Use Oh-My-Zsh
	- Just what I used, and I based this on my testing
- Look into X11 forwarding and XQuartz for terminals
	- Wasn't necessary in my case but might be in other cases

Partially based on:
https://nitin-sharma.medium.com/setting-up-cooja-in-linux-machine-20b25cea370d
https://anrg.usc.edu/contiki/index.php/Installation
https://docs.contiki-ng.org/en/develop/doc/getting-started/Docker.html
