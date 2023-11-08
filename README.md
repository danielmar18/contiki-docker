# Instructions for Set-up and usage of Contiki and Cooja
This process was originally created for the M1/M2/M3/etc Macbooks but in theory should also work with the Intel-based Mac's with some minor tweaks to the steps. The general theory shoudl also work for Windows but will need a seperate instruction list.

**Support:**

**Apple Silicon Processors (M1, M2, etc.)** :white_check_mark:

**Apple Intel-based Processors** :eight_pointed_black_star:

**Windows / Linux** :question:

I highly recommend going through the entire instruction list once before starting the process, at least for the Contiki & Docker part. The "Contiki Setup" goes through the actual installation and setup process while "Host v. Docker Workflow" explains exactly how to get the best out the cohabitation of Docker and local terminal. 

This instruction set was made for Apple Macbooks using Silicon processors but should in theory be partly useful for other OS's as well, at least the Contiki Setup itself.
## Contiki Setup
1. Install [Docker](https://docs.docker.com/desktop/) & Git
	1. Sign In/Up in Docker Application
2. **IMPORTANT:** Install the MSP430 Compiler 
	1. Use the commands from the Vagrant bootstrap shellscript to install the compiler needed for `TARGET=sky`. It won't be used but the Contiki `make` command needs to be able to locate it.
	2. Open a terminal and run these commands:
		1. `wget http://simonduq.github.io/resources/mspgcc-4.7.2-compiled.tar.bz2`
		2. `tar xjf mspgcc*.tar.bz2 -C /tmp/`
		3. `sudo cp -f -r /tmp/msp430/* /usr/local/`
		4. `rm -rf /tmp/msp430 mspgcc*.tar.bz2`
3. Create directory, move into it(`cd <dir_name>`), clone Git repo and move into it (`cd contiki-ng`)
	1. git clone [https://github.com/contiki-ng/contiki-ng.git](https://github.com/contiki-ng/contiki-ng.git) --branch release/v4.9
	2. **IMPORTANT**: Move inside the `contiki-ng` directory before next step
4. Add the Contiki absolute path to an environmental variable
	1. `export CNG_PATH=<absolute-path-to-your-contiki-ng>`
	2. Can be found by running `pwd`
5. Create `contiker` alias for running the Docker image
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
<img width="1149" alt="image" src="https://github.com/danielmar18/contiki-docker/assets/33004159/61038b2d-3b27-4e65-af52-02a9faf85013">

### Docker/Contiki Troubleshoot
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
- Credit: [zemendes1](https://github.com/zemendes1). If you get the error `Make is older than version 4.0.0` run the following:
	- `brew install make`
	- Then add the following line to ~/.zshrc or ~/.bashrc depending on terminal setup:
		- `export PATH="/usr/local/opt/make/libexec/gnubin:$PATH”`

## Host v. Docker Workflow
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
##### No Vagrant, No Virtual Machine!

## Cooja Setup
1. Install [OpenJDK@17](https://formulae.brew.sh/formula/openjdk@17#default) through Homebrew
	1. **Important**: Has to be version 17 if used with release 4.9 or so it seems
		1. Depending on Homebrew setup, you might get an error along the lines of `cannot install in Homebrew on ARM processor in Intel default prefix`. If so, add `arch -x86_64` before `brew install...` command
	2. Add package location to `PATH` and `JAVA_HOME`:
		1. Installation process will usually recommend the easiest way to add to the variables. If not, run `brew info openjdk@17` to find package location
		2. **NOTE:** These paths will depend on Homebrew setup! If it won't work with the version number(`<PATH>/openjdk@17/17.X.X`), try exporting without it(`<PATH>/openjdk@17`).
		3. Run `export PATH=${$PATH}:/<PACKAGE_LOCATION>/bin`.  E.g:
			1. `export PATH=${$PATH}:/opt/homebrew/opt/openjdk@17/bin
			2. `export PATH=${$PATH}:/usr/local/Cellar/openjdk@17/17.0.9/bin
		4. Run `export JAVA_HOME=/<PACKAGE_LOCATION>`. E.g:
			1. `export JAVA_HOME=/opt/homebrew/opt/openjdk@17
			2. `export JAVA_HOME=/usr/local/Cellar/openjdk@17/17.0.9
2. Move into `contiki-ng/tools/cooja` directory
	1. Run command `git submodule update --init` to install necessary code for Cooja to run
		1. **This will take some time to run. Get a cup of coffee, you deserve it!**
3. Once the command has finished, run `./gradlew run`in the same directory, and Cooja should start right up!
	1. If you compiled a `.sky` file in previous step, try and import into a simulation!

Same as with Contiki, with a bit of Homebrew and environmental path magic, we get a much simpler and more enjoyable process. Both Cooja and Mote development in the same directory, without any VM hassle.
###### No Vagrant, No Virtual Machine!
### Cooja Troubleshooting
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

Special thanks to the alpha tester [zemendes1](https://github.com/zemendes1)for pointers on the process and instructions!
