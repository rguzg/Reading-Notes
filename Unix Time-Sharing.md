# The UNIX Time-Sharing System
https://chsasank.github.io/classic_papers/unix-time-sharing-system.html

*Writers notes are in itallics*

- The design choices described in this paper significantly influenced the modern OS design

## What is Unix?
- General-purpose, multi-user interactive OS
- Has features not available in other OSs *(at least in 1974)*:
    - Hierarchical file system w/ demountable volumes
    - Compatible file, device and inter-process I/O
    - Asynchronous processes
    - A bunch of subsystems
- Unix can run on hardware costing as little as $40k *(lul)*
- One of the included programs in Unix was a compiler for a "language resembling BCPL with types and structures AKA **C**

## Hardware and Software Environment
- Unix takes a big *(for the time)* 42KB of RAM
- That big memory footprint is because it includes device drivers and an allotment of space for I/O buffers and system tables *(not really sure what those are)*
- Most of Unix is written in C

## File System
Files are super important for the Unix architecture. **Unix exposes all funcionality through files.** **From the point of the user** there're three types of files:


### Ordinary Files
- Stores whatever text or binary data the user might want to store
- No particular structuring is expected by the system *(Does this mean that this types of files can be stored anywhere?)*
- Text files are **strings of characters, with lines demarcated by the new-line character.** *(The famous '\n')*
- Binary programs are **sequences of bytes as they will appear in core memory when the program starts executing**
- Programs can create their own files with a more detailed structure
- Files are named by sequences of 14 or fewer characters

### Directories
- They provide a mapping between the names of files and the files themselves
- Directories give structure to the file system
- Each user has its own directory. Users can create subdirectories inside that directory
- A directory cannot be written on by unprivileged programs. The OS controls the contents of the directories *(I'm assuming there're APIs so that programs can store files inside folders)*
- **Root Directory**:
    - All files can be found by tracing a path through a chain of directories
    - The starting point for file searches is the root directory
- **Commands Directory**: Contains all the programs provided for general use
- **Path names**: Sequence of directory names separated by slashes and ending in a file name. Path names can be used when especifing the name of a file to the system
    - If the path name starts with a **/** the system begins searching in the root directory
    - If the path name doesn't start with a **/** the system begins searching in the current directory
    - A name without any **/** refers to a file inside the current directory
    - The **null** file refers to the current directory *(ls NULL?)*
- **Linking**: The same nondirectory file can appear in several directories under different names.
    - A file does not exist within a particular directory; the **directory entry consists of its name and a link to the actual file**
    - Files exist independently of directories (although if you delete all links to a file, the file also disappears)
- Directories always have two entries:
    - *.* refers to the current directory
    - *..* refers to the parent of the current directory
- **Rooted tree structure**: 
    - Each directory must appear as an entry in exactly one other directory (Except fot the special entries . and ..)
    - This avoids the separation of portions of the directory hierarchy and simplifies the writing of programs that navigate the directory structure
    - *The rooted tree structure is **so** engrained into the way file systems work that I can't imagine how folder navigation would work if the rooted tree structure didn't exist. Imagine trying to find folders that are in random places in the file system!*

### Special Files
- These are the weird part of the UNIX file system (then CP/M (and thus MS-DOS) stole the idea and the concept became less weird)
- Each I/O device supported by Unix is associated with at least one of these files
- These files are read and written like any ordinary file, but requests to read or write result in activation of the associated device
- These files are stored in **/dev**, although links may be made to these files
- To punch paper tape, for example *(this example is totally valid in this day and age! /s)*, you just write on the /dev/ppt file
- Special files exist for **comunication lines, disks, drives and physical memory**
- Active disks and other core special files are protected from indiscriminate access
- There are three main advantages in trating I/O devices this way:
    - File and device I/O are as similar as possible
    - File and device names have the same syntax and meaning (if a program is expecting a file name, you can also pass a device name)
    - Special files are subject to the same protections as regular files
