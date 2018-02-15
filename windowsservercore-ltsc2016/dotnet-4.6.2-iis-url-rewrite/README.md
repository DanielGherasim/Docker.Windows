## Windows Server Core LTSC2016 + .NET 4.6.2 + IIS + URL Rewrite

This image is a merge between the following 2 Dockerfiles:
* https://github.com/Microsoft/aspnet-docker/blob/master/4.6.2-windowsservercore-ltsc2016/runtime/Dockerfile
* https://github.com/Microsoft/dotnet-framework-docker/blob/master/4.6.2-windowsservercore-ltsc2016/runtime/Dockerfile

The following custom configurations were added:
1. Fixed DNS on the Container until it will be fixed (in 1709 Creators Update - https://github.com/moby/moby/issues/27499).
1. Installed URL Rewrite.
1. Added DefaultAppPool to the Administrators Group.

The image runtime script is tested with Docker EE 17.06 on Windows Server 2016 host.
