# dotnet-shared-host
Guide to install dotnet without root / sudo on a shared host like DreamHost.


Does your web hosting provider refuse to provide dotnet core / dotnet 6 / dotnet 7 support?

Are you using DreamHost shared hosting, but would like to use c# instead of php?

Many hosting providers are living in the past - refusing to allow ASP.NET, not realizing the .NET Framework is now completely open source, cross platform and has the capacity to perform significantly better than rubbish like PHP, Python or *VOMIT* node.js.

It is bigotry against Microsoft, pure and simple. Irrational and foolish.

What can you do though? Installing and using dotnet requires root or sudo privs...

## The steps

Microsoft provides a manual installer script that works without root / sudo: [dotnet-install scripts reference](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-install-script)
Presently, the script can be downloaded here: (https://dot.net/v1/dotnet-install.sh)

If you install the sdk, however, you will get the following error when you try to do ANYTHING with the dotnet executable:

```Failed to create CoreCLR, HRESULT: 0x80004005```

If you install only the runtime you will encounter it less. The sdk automatically creates some files in the root /tmp directory - which we obviously can't touch. setting TMPDIR doesn't work. Microsoft added an environment variable: [Add environment variable (COMPlus_EnableDiagnostics) to disable debugging and profiling](https://github.com/dotnet/coreclr/pull/15878)

```export COMPlus_EnableDiagnostics=0```

This should be added to the beginning of .bashrc, .bash_profile or the startup script for whichever shell you use.

This alone unfortunately doesn't handle all scenarios - shared hosts have more security lock-downs (for good reason) that the dotnet executable explodes against.

One more step:
[.NET 5 application on Linux fails to create CoreCLR, 0x80004005](https://github.com/dotnet/runtime/issues/46462#issuecomment-752563547) - huge thanks to @janvorli for figuring this out!

```paxctl -c -m ~/.dotnet/dotnet```

This patches the executable and seems to resolve the issue!

If your host doesn't have paxutil, you must download and extract the appropriate version for your host's distro. Here is an example that worked with Ubuntu 18.04.

```
 wget http://launchpadlibrarian.net/363336646/paxctl_0.9-1build1_amd64.deb
 ar -x paxctl_0.9-1build1_amd64.deb
 tar -xvf data.tar.xz
 cd sbin
 paxctl -c -m ~/.dotnet/dotnet
```

You should add the following to ~/.bash_profile or ~/.bashrc - at the beginning, before any checks for interactive sessions:

```
export DOTNET_ROOT=$HOME/.dotnet
export PATH=$PATH:$HOME/.dotnet:$HOME/.dotnet/tools
export COMPlus_EnableDiagnostics=0
```

After patching dotnet, you should be able to create, build and run projects!

* TODO: Write a script to do it automatically
* TODO: How to handle updates
* TODO: Getting apache to reverse-proxy ASP.NET
