#Cross Compilation for the Zynq Platform
While the Zynq-7000 series platform is a powerful one, it can become quite tedious to develop on the platform using SSH and the sometimes unreasonably slow build times.
This guide will go through the process of setting up a Linux environment for cross compilation.
A small program is made available to assist with some of the concepts in this project.

##Setting up the environment
Xilinx provides a number of compilers with the Vivado installation. Which one to use depends on the platform to be compiled for. In the case of the Linux environment for the [MicroZed](https://github.com/SDU-Embedded/linux_zynq/wiki/Installing-Linux-on-the-MicroZed) or [Zybo](https://github.com/SDU-Embedded/linux_zynq/wiki/Installing-Linux-on-the-ZYBO) available here at SDU-Embedded, `arm-linux-gnueabihf-gcc` should be used. In order to access this, source the following in a terminal:
	
	source path/to/vivado/2016.4/settings64.sh

Running this command will add a number of Xilinx related paths to `$PATH`, among them the path to all of the provided compilers. Retyping this command becomes tedious reasonably fast and the recommended approach is to add the above line to the `.bashrc` file with:

	echo "source path/to/vivado/2016.4/settings64.sh" >> ~/.bashrc

##Compiling a Program
Once the above step is done, the cross compiler can be used in the same manner as gcc:
	
	arm-linux-gnueabihf-gcc -o hello hello.c -I ../include/

It is likely that, as a project evolves, more and more files and flags will have to be added to this command. `make` can be used to assist in making the build process less tedious.

Throughout the remainder of this guide, reference will be made to a [simple c program]() which was made to assist in presenting some of the concepts. The folder structure of that program is relevant to the process and is shown here for convenience.

	.
	+-- Makefile
	+-- bin
	|	+-- hello
	+-- include
	|	+-- hello.h
	+-- src
	|	+-- hello.c
	|	+-- main.c

Every build file is written to the `bin/` folder, every header file to the `include/` folder and every source file to the `src/` folder.

##Creating a Makefile

This is the Makefile, also present in the root folder of the hello project. While this is probably not sufficient for every project, it should at least provide a good starting point that can later be changed to fit the needs of the project. The functionality is explained throughout the remainder of the guide.
	
	CC:=gcc
	CROSS_COMPILER:=arm-linux-gnueabihf-

	TARGET:=hello

	SOURCEDIR:=.
	BUILDDIR:=bin/
	INCLUDEDIR:=include/

	SOURCES := $(shell find $(SOURCEDIR) -name '*.c')
	INCLUDE := $(shell find $(SOURCEDIR) -name '*.h')
	OBJECTS := $(SOURCES:%.c=%.o)

	CFLAGS :=

	%.o: %.c $(INCLUDE)
		$(CROSS_COMPILER)$(CC) $(CFLAGS) -c -o $@ $< -I $(INCLUDEDIR)

	$(BUILDDIR)$(TARGET): $(OBJECTS)
		$(CROSS_COMPILER)$(CC) -o $@ $^

	.PHONY: clean
	clean:
		rm $(TARGET) $(OBJECTS)

The initial two lines define the compiler to use. Compiling for a different platform can be done by changing `CROSS_COMPILER` accordingly. `TARGET` is the name of the output file and `BUILDDIR` and `INCLUDEDIR` define the build- and header-directories, respectively. `SOURCEDIR` is the root folder of the project and can be left as `.` if `make` is called from the root folder. The lines:
	
	SOURCES := $(shell find $(SOURCEDIR) -name '*.c')
	INCLUDE := $(shell find $(SOURCEDIR) -name '*.h')

Creates a list of all header and source files by using the `$(shell <cmd>)` directive. This will execute the given command and return the result, in this case `find` is used to return every file with the `.c` and `.h` extension.

	OBJECTS := $(SOURCES:%.c=%.o)

This is called [substitution references](https://www.gnu.org/software/make/manual/make.html#Substitution-Refs) and will take every element in sources with a `.c` extension and replace it with the `.o` extension.
	
	CFLAGS := 

In this very simple project, no flags are necessary beyond the basics. Should the programmer wish to add compile flags it can be done by setting in this variable.

	%.o: %.c $(INCLUDE)
		$(CROSS_COMPILER)$(CC) $(CFLAGS) -c -o $@ $< -I $(INCLUDEDIR)

This is where things start to get interesting! Firstly, the format for any makefile is as such:

	targets : prerequisites
		recipe

Targets is which files the rule acts upon. Prerequisites are those files that should be present before the rule can be executed and the recipe is what should happen to that target.

Now back to the example, Make can only detect if changes are made to the `.c` files since it does not parse the links between files. For this reason, it is necessary, or at least preferable, to set up a rule that makes Make aware of these changes. These lines say that for any `.o` file, recompile it if either that `.c` file or any of the files in the `$(INCLUDE)` variable has a timestamp more recent than it. The compilation is done using the -c flag. This flag indicates that the source files should be compiled, but not linked, resulting only in an object file as the output, which is used in the actual build step. Note that the line above makes use of two macros: `$@` is shorthand for the target name and `$<` is shorthand for the first prerequisite. 

	$(BUILDDIR)$(TARGET): $(OBJECTS)
		$(CROSS_COMPILER)$(CC) $(CFLAGS) -o $@ $^

This is the actual build step in the Makefile. It creates, or links, all of the `.o` files created using the previously described rule. Here, the compiler is called with the macro `$^` which is shorthand for every prerequisite.

	.PHONY: clean
	clean:
		rm $(TARGET) $(OBJECTS)

Finally, a rule to make cleaning up the project folder simple. The `.PHONY` directive tells make that this rule should not result in an output file. It is not strictly necessary but could potentially save some headaches. Next, the recipe simply empties the build folder and removes any object file from the `src/` folder.