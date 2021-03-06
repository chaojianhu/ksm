# ksm v1.3 [![Build Status](https://travis-ci.org/asamy/ksm.svg?branch=master)](https://travis-ci.org/asamy/ksm) [![Build Status](https://ci.appveyor.com/api/projects/status/nb7u22qxjabauex5?svg=true)](https://ci.appveyor.com/project/asamy/ksm)

A really simple and lightweight x64 hypervisor written in C for Intel processors.

KSM aims to be fully feature fledged and as general purpose as possible,
although there are absolutely no barriers, even extending it to be a
multi-purpose thing is perfeclty fine, e.g. a sandbox, etc.

Currently, KSM supports Windows and Linux kernels natively, and aims to support
macOS by 2017, if you want to port KSM see porting guidelines down below.  Note
that the `master` branch may be unstable (bugs, unfinished features, etc.), so
you might want to stick with the releases for a captured stable state.

## Features

- IDT Shadowing
- EPT violation #VE (if not available natively, VM-exit path is taken)
- EPTP switching VMFUNC (if not available natively, it will be emulated using a VMCALL)
- APIC virtualization
- VMX Nesting

## Why not other hypervisors?

You may have already guessed from the `Features` part, if not, here are some reasons:

- Do not implement the new processor features KSM implements (VMFUNC, #VE, etc.)
- Are not simple enough to work with or understand
- Simply, just have messy code base or try too hard to implement endless C++ features that just make code ugly.
- Too big code base and do not have the same purpose (e.g. research or similar)

Such features for such purpose is really crucial, for my purpose, I wanted a quicker physical memory virtualization
technique that I can relay on.

## Requirements

- An Intel processor (with VT-x and EPT support)
- A working C compiler (GCC or CLang or Microsoft compiler aka CL).  CLang may
not be able to compile it and is not tested.

## Supported Kernels

- All x64 NT kernels starting from the Windows 7 NT kernel.  It was mostly tested under Windows 7/8/8.1/10.
- Linux kernel (tested under 3.16, 4.8.13 and mainline)

## Porting to other kernels guidelines

- Port `mm.h` functions (`mm_alloc_page`, `__mm_free_page`, `mm_alloc_pool`,
			 etc.)
- Port `resubv.c` (not really needed) for re-virtualization on S1-3 or S4 state (commenting it out is OK).
- Write module for initialization
- Port `print.c` for printing interface (Some kernels may not require it)
- Port `vmx.S` for the assembly based stuff, please use macros for calling conventions, etc.

Hopefully didn't miss something important, but these are definitely the mains.

## Porting to other x86 processors

Since some code is split oddly, and needs to be organized to fit logically
together, these files should be renamed/merged:

- exit.c and vcpu.c should be merged to make vmx.c
- functions from ksm.c may need to be merged into vmx.c as #1

## TODO / In development

- APIC virtualization (Partially implemented, needs testing & fixes)
- TSC virtualization
- UEFI support
- Intel TXT support
- AMD-V with NPT support
- Nesting support (Some fixes needed and support for minor features)
- More documentation
- Finish writing tests
- Failsafe state (e.g. when an unexpected thing happens, turn off and restore
                  state to a valid one.)

See also Github issues.  Some of these features are unfortunately not
(fully) implemented due to lack of hardware (support) or similar.

## KSM needs your help to survive!

Contributions are really appreciated and can be submitted by one of the following:

- Patches (e-mail)
- Github pull requests
- git request-pull

It'd be appreciated if you use a separate branch for your submissions (other
								       than
								       master,
								       that
								       is).

The github issues is a great place to start, although implementing new features
is perfectly fine and very welcome, feel free to do whatever your little heart
wants.

The following is _not_ required, but **prefered**:

1. Put your copyright on top of the file(s) you edit along with a tiny description
with your changes.  Something like:

```c
/*
   ...
   Copyright (C) 2016 Your Name <your_email@domain.com>
	- Added support for XXX
	- Fixed bug with YYY
   ...
 */
```

2. Format your git commit messages properly (A signed-off-by is good but
   **not** required, note: you can use `git commit --signoff` instead of writing
   manually.  See also Linux kernel contribution guidelines for more perks):
```
vmx: fix issue with xxx

Write as much as you would like as needed or point to some issue, although
writing is prefered, or even comments in the code itself is much better.

Optional:
Signed-off-by: Your Name <your_email@domain.com>
```

### Setting up your git tree

For the sake of simplicity, we're going to use some names placeholders (which
									in
									reality
									you
									should
									replace
									with
									your
									own):

1. `LOCAL_BRANCH` - is your local branch you're going to be committing to (e.g.
   `my-changes`).
2. `REMOTE_BRANCH` - is the branch name you have in your remote repository (e.g.
   `pull-me`, can be the same as `LOCAL_BRANCH`).
3. `REMOTE_URL` - Your remote repository URL (e.g. https://github.com/XXX/ksm).

	Note: you do not have to have a remote repository, you can commit to
	your local copy, then just use patches, see below.
4. `USER_NAME` - Your username

Get to it:

1. Get a local copy: `git clone git@host.com:name/ksm`
2. Switch to a new branch: `git checkout -b LOCAL_BRANCH`
2. Setup remote: `git remote add upstream https://github.com/asamy/ksm` (Can be
skipped)
3. (When there is a change in my tree) Pull my tree: `git pull --rebase upstream master` (If #2 is skipped, then use
						     the complete URL in place
						     of `upstream`.)  You can
also use `git rebase -i upstream/master` to rebase your commit(s) on top of my
new changes, but pulling is better and will also rebase.
4. Commit something: `git commit -a --signoff -m "commit message"` (Signing off
   commits is optional, you can also sign with PGP, but if you're going to
   submit patches, then the PGP signature is going to get purged.)

#### Submitting your changes

If you're going to use patches, then simply:

`git format-patch HEAD~X`

Where X is the number of commits (patches) to create, can be ommitted if 1
commit only, e.g.:

`git format-patch HEAD~`

You can then use the patch file(s) as an attachment and e-mail them manually, or
you can use git SMTP with `git send-email` to do it for you.

##### Using pull requests

You have 2 options (if using 1st, then skip the rest):

1. If you're using github fork, you can just use the github pull request
   interface.
2. If you're going to use git request-pull follow.

###### Using git-request-pull

Usage:

`git request-pull START_COMMIT REPOSITORY_URL END_COMMIT`

First publish your changes:

`git push origin REMOTE_BRANCH`

To summarize a branch changes:

`git request-pull abcd https://github.com/USER_NAME/ksm HEAD`

Which will summarize changes from commit `abcd` to `HEAD` of which you can then
e-mail me that summary.

You can also use:

`git request-pull master https://github.com/USER_NAME/ksm
LOCAL_BRANCH:REMOTE_BRANCH`

Which will summarize changes from the local master branch (Which should contain
							   my changes, i.e. my
							   tree) to your
changes.

`REMOTE_BRANCH` can be omitted if same as `LOCAL_BRANCH`.
You can also specify a tag of your choice, in that case, use tag names instead
of commit hashes/branch names.

###### Request a pull manually

Simply e-mail me the remote and branch of which to pull from, e.g.:

```
...
Please pull from https://github.com/USER_NAME/ksm REMOTE_BRANCH
...
```

It's however, much better if you use `git request-pull` to automatically
summarize changes.

## Building

### Building for Linux

Install kernel headers:

- Debian/Ubuntu: `[sudo] apt-get install linux-headers-$(uname -r)`
- ArchLinux: `[sudo] pacman -S linux-headers`
- Fedora: `[sudo] yum install kernel-devel kernel-headers`

Then `make`.

### Building for Windows

#### Compiling under MinGW

##### Makefile variables:

1. `C=1` - Prepare for cross-compiling.
2. `V=1` - Verbose output (the default, pass 0 for quiet.)

##### Cross under Linux

Install the following packages:

- Debian/Ubuntu: `[sudo] apt-get install gcc-mingw-w64-x86-64
binutils-mingw-w64-x86-64`
- ArchLinux: `[sudo] pacman -S mingw-w64-gcc`
- Fedora: `[sudo] yum install mingw64-gcc`

Then `make -f Makefile.windows C=1`

##### Under Native

Natively, you'll want to adjust (or pass in command line) DDK paths, e.g.:

`mingw32-make -f Makefile.windows CROSS_INC=/path/to/include/ddk`

Or, simply just edit Makefile.windows manually.  Also make sure to adjust your
environment variables (PATH) to point to the right `bin/` directory where the
compiler, etc lie.

#### Compiling under MSVC

The solution under `ksm/` directory is a VS 2015 solution, you can use it to build, you'll
also need the Windows Driver Development Kit.

To build from VS command line, simply cd to where `ksm` is and:

`msbuild ksm\ksm.sln`

Or:

`msbuild ksm\ksm\ksm.vcxproj`

## Loading the driver

### On Linux
Loading:
- `sudo make load`  

Unloading:
- `sudo make unload`

Output:
- `sudo dmesg -wH`

### On Windows
In commandline as administrator:

1. `sc create ksm type= kernel binPath= C:\path\to\your\ksm.sys`
2. `sc start ksm`

Unloading:
- `sc stop ksm`

You can also use [kload](https://github.com/asamy/kload)  
Output can be seen via DebugView or WinDBG if live debugging (You might want to
							      execute `ed
							      Kd_DEFAULT_Mask
							      8`).

## Some technical information

### Some notes

To simplify things, the following terms are used as an abbreviation:

1. Host - refers to the VMM (Virtual Machine Monitor) aka VMX root mode
2. Guest or Kernel - refers to the running guest kernel (i.e. Windows or Linux)

Some things need to be used with extra care especially inside Host as
this is a sensitive mode and things may go unexpected if used improperly.

- The timestamp counter does not _pause_ during entry to Host, so
things like APIC timer can fire on next guest entry (`vmresume`).
- Interrupts are disabled.  On entry to `__vmx_entrypoint`, the CPU had already
disabled interrupts.  So, addresses referenced inside root mode should be
physically contiguous, otherwise if you enable interrupts by yourself, you
might cause havoc if a preemption happens.
- Calling a Kernel function inside the Host can be dangerous, especially
because the Host stack is different, so any kind of stack probing
functions will most likely fail.
- Single stepping `vmresume` or `vmlaunch` is invaluable, the debugger will
never give you back control, for obvious reasons.  If you want that behavior,
      then rather set a breakpoint on whatever `vcpu->ip` is set to.
- Virtualization Exceptions (#VE) will not occur if:
	1. The processor is delivering another exception
	2. The `except_mask` inside `ve_except_info` is set to non-zero value.
- If the processor does not support Virtualization Exceptions, the VM exit path
will be taken instead (Note that the VM exit path is _always_ handled).
- If the processor does not support VMFUNC, it's emulated via VMCALL instead.

### Debugging and/or testing

Since #VE and VMFUNC are now optional and will not be enabled unless the CPU support it,
you can now test under VMs with emulation for VMFUNC.

#### Live debugging under Windows

You may want to disable `SECONDARY_EXEC_DESC_TABLE_EXITING` in vcpu.c in secondary controls,
otherwise it makes WinDBG go **maniac**.  I have not investigated the root cause, but it keeps
loading GDT and LDT all the time, which is _insane_.

### IDT shadowing

- By enabling the descriptor table exiting bit in processor secondary control, we can easily establish this
- On initial startup, we allocate a completely new IDT base and copy the current one in use to it (also save the old
												   one)
- When a VM-exit occurs with an `EXIT_REASON_GDT_IDT_ACCESS`, we simply just give them the cached one (on sidt) or (on
														  lidt),
	we copy the new one's contents, discarding the hooked entries we know about, thus not letting them know about
	our stuff.

### #VE setup and handling

We use 3 EPT pointers, one for executable pages, one for readwrite pages, and last one for normal usage.  (see next
													   section)

- `vcpu.c`: in `setup_vmcs()` where we initially setup the VMCS fields, we then set the relevant fields (`VE_INFO_ADDRESS`,
													`EPTP_LIST_ADDRESS`,
													`VM_FUNCTION_CTL`) and enable
relevant bits VE and VMFUNC in secondary processor control.

- `vmx.asm` (or `vmx.S` for GCC): which contains the `#VE` handler (`__ept_violation`) then does the usual interrupt handling and then calls
	`__ept_handle_violation` (`vcpu.c`) where it actually does what it needs to do.
- `vcpu.c`: in `__ept_handle_violation` (`#VE` handler *not* `VM-exit`), usually the processor will do the `#VE` handler instead of
	the VM-exit route, but sometimes it won't do so if it's delivering another exception.  This is very rare.
- `vcpu.c`: while handling the violation via `#VE`, we call `vmfunc` only when we detect that the faulting address is one of
	our interest (e.g. a hooked page), then we determine which `EPTP` we want and execute `VMFUNC` with that EPTP index.

### Hooking executable pages

#### Execute-only EPT for executable page hooking, RW for read or write access

	(... to avoid a lot of violations, we just mark the page as execute only and replace the _final_ page frame
	 number so that it just goes straight ahead to our trampoline)
Since we use 3 EPT pointers, and since the page needs to be read and written to sometimes (e.g. patchguard
											   verification),
      we also need to catch RW access to the page and then switch the EPTP appropriately according to
      the access.  In that case we switch over to `EPTP_RWHOOK` to allow RW access only!
	The third pointer is used for when we need to call the original function.  The third pointer
	has execute only access rights to the page with the sane page frame number.

## Enabling certain features / tests

You can define one or more of the following:

- `EPAGE_HOOK` - Enables executable page shadow hook
- `ENABLE_PML` - Enables Page Modification Log if supported.
- `EMULATE_VMFUNC` - Forces emulation of VMFUNC even if CPU supports it.
- `EPT_SUPPRESS_VE` - Force suppress VE bit in EPT.
- `ENABLE_RESUBV` - Enable S1-3-S4 power state monitoring for re-virtualization
- `NESTED_VMX` - Enable experimental VT-x nesting
- `ENABLE_FILEPRINT` - Available on Windows only.  Enables loggin to
disk
- `ENABLE_DBGPRINT` - Available on Windows only.  Enables `DbgPrint`
log.

## Reporting bugs (or similar)

You can report bugs using Github issues, please provide the following:

- System information (version including build number, CPU information perhaps codename too)
- The git tree hash
- KSM output if available
- Kernel Du Jour
- Anything else you feel is relevant

If it's a crash, please provide the following:

### For Windows

- A minidump (C:\windows\minidump) or a memory dump (C:\windows\memory.dmp).  Former prefered.
- The compiled .sys and the .pdb/.dbg file
- The Kernel executable if possible, e.g. ntoskrnl.exe from C:\Windows\System32

### For Linux

- `ksmlinux.ko` and `ksmlinux.o`
- Stack dump from dmesg or kernel panic

## References

- Linux kernel (KVM)
- HyperPlatform
- XEN

## License

GPL v2 firm, see LICENSE file.  Note that some code is thirdparty, respective
licenses and/or copyright should be there, if you think it's not, please let me
know.  Most of the code is GPL'd, though...

