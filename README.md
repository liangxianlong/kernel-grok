# kernel-grok
grok the linux kernel via a cmake shim

##

I'm sure there are great ways to spelunk the kernel with an ide... This is my attempt to rig something together using a simple ruby script and a roll of duct tape. 

## Nutshell: 
  * create `compile_commands.json` for kernel using an intercepted build
  * use ruby script to convert `compile_commands.json` into an IDE friendly `CMakeLists.txt`

## Possible steps
(Note: not fully tested.  Requires some prior knowledge to work around typos etc)


* Install [Bear](https://github.com/rizsotto/Bear) (A build interceptor)
```bash
git clone https://github.com/rizsotto/Bear && cd Bear
mkdir build && cd build
cmake .. && make 
sudo make install
cd ../..
``` 

* Grab the kernel and build
```bash
git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
cd linux-stable
git reset --hard v4.14-rc2  ## random known good version
make defconfig

## Note that we use bear to intercept the build
bear make -j 12  ## create compile_commands.json
cd .. 
```

* Use our ruby script 
```  
git clone https://github.com/habemus-papadum/kernel-grok.git

cd linux-stable
../kernel-grok/generate-cmake  ## creates CMakeLists.txt

## test that generated cmake is valid
mkdir build
cd build
cmake .. && make -j12
```

* Now open linux-stable/CMakeLists.txt in an ide (tried testing with clion; though takes about 10 minutes to index the project initially; at first it seemed like an epic fail, but seems quite usable after initial indexing and then restarting)
* note `generate-cmake -d` option if not using clion or if cmake build directory is not one level under
  kernel source root 

## Theory of operation
* using compilation database generated by `bear`, a cmake shim is created that creates a single
  cmake `OBJECT` library (this is a somewhat obscure library type that as opposed to `SHARED` or `STATIC` performs no linking).
* The shim should properly properly compile.  This means your ide can be used not only for semantic navigation e.g. it knows where to find headers and knows about -Ddefines and so can jump to defintions and find usages etc, but also allows for testing whether edits to existing files compile -- perhaps 90% of the typical dev workflow.
* Analogy might be Xcode projects generated by CMake.  Can be used to make some edits, but larger things like adding files, etc cannot be done in Xcode.  So after big changes e.g. new file, new kernel config,  you must repeat KBuild intercepted via bear, regenerate shim, and then reload project in ide
* Use ide compilation to test if your edits compile.  Use KBuild to create a proper image to run in qemu etc.  
* NOTE: Everything depends on the kernel actually being built with normal kbuild.  There are generated files that are needed.  
* NOTE: Although a CMakeLists.txt is included in the repo for providing a quick look at what is being done, it should not be used -- GENERATE YOUR OWN
* NOTE: linux assumes builds are done from the root src directory e.g. "in-tree".  Some hacking is done to allow the cmake shim to work out of tree (again, cf. `-d` option as well as the actual ruby code (which is pretty simple)) 
* A similar approach can be taken to spelunk through other non-cmake projects , though each project will require some tinkering

      
  
   
