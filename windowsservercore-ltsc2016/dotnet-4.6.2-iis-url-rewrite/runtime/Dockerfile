# escape=`

# This runtime script is only tested with Windows Server 2016 and Docker EE 17.06
# Builds ASP.NET Ready container with .NET up to 4.6.2 installed on it.
# For applications which are not tested running on higher .NET versions than .NET 4.6.2.

# The `FROM` instruction specifies the base image. You are extending the `microsoft/windowsservercore:ltsc2016` image.

# 1. https://github.com/Microsoft/dotnet-framework-docker/blob/aspnet-build/4.6.2-windowsservercore-ltsc2016/runtime/Dockerfile

FROM microsoft/windowsservercore:ltsc2016

# Install .NET Framework 4

RUN set COMPLUS_NGenProtectedProcess_FeatureEnabled=0 & `
	\Windows\Microsoft.NET\Framework64\v4.0.30319\ngen update & `
	\Windows\Microsoft.NET\Framework\v4.0.30319\ngen update
	
# 2. https://github.com/Microsoft/aspnet-docker/blob/master/4.6.2-windowsservercore-ltsc2016/runtime/Dockerfile

# USE powershell and set command parameters for it

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Enable IIS and add ServiceMonitor Features on the Container

RUN Add-WindowsFeature Web-Server; `
	Add-WindowsFeature NET-Framework-45-ASPNET; `
	Add-WindowsFeature Web-Asp-Net45; `
	Remove-Item -Recurse C:\inetpub\wwwroot\*; `
	Invoke-WebRequest -Uri https://dotnetbinaries.blob.core.windows.net/servicemonitor/2.0.1.1/ServiceMonitor.exe -OutFile C:\ServiceMonitor.exe
	
# Download Roslyn nupkg and ngen the compiler binaries

RUN Invoke-WebRequest https://api.nuget.org/packages/microsoft.net.compilers.2.3.1.nupkg -OutFile c:\microsoft.net.compilers.2.3.1.zip ; `	
    Expand-Archive -Path c:\microsoft.net.compilers.2.3.1.zip -DestinationPath c:\RoslynCompilers ; `
    Remove-Item c:\microsoft.net.compilers.2.3.1.zip -Force ; `
    &C:\Windows\Microsoft.NET\Framework64\v4.0.30319\ngen.exe install c:\RoslynCompilers\tools\csc.exe /ExeConfig:c:\RoslynCompilers\tools\csc.exe | `
    &C:\Windows\Microsoft.NET\Framework64\v4.0.30319\ngen.exe install c:\RoslynCompilers\tools\vbc.exe /ExeConfig:c:\RoslynCompilers\tools\vbc.exe  | `
    &C:\Windows\Microsoft.NET\Framework64\v4.0.30319\ngen.exe install c:\RoslynCompilers\tools\VBCSCompiler.exe /ExeConfig:c:\RoslynCompilers\tools\VBCSCompiler.exe | `
    &C:\Windows\Microsoft.NET\Framework\v4.0.30319\ngen.exe install c:\RoslynCompilers\tools\csc.exe /ExeConfig:c:\RoslynCompilers\tools\csc.exe | `
    &C:\Windows\Microsoft.NET\Framework\v4.0.30319\ngen.exe install c:\RoslynCompilers\tools\vbc.exe /ExeConfig:c:\RoslynCompilers\tools\vbc.exe | `
    &C:\Windows\Microsoft.NET\Framework\v4.0.30319\ngen.exe install c:\RoslynCompilers\tools\VBCSCompiler.exe  /ExeConfig:c:\RoslynCompilers\tools\VBCSCompiler.exe ;

# Set Environmnet Variable for the Roslyn Compyler Location
	
ENV ROSLYN_COMPILER_LOCATION c:\\RoslynCompilers\\tools

# Set Service Monitor asn the Main ENTRYPOINT of the Container

ENTRYPOINT ["C:\\ServiceMonitor.exe", "w3svc"]

# 3. Custom

# Fix DNS on the Container until it will be fixed (in 1709 Creators Update I guess) - https://github.com/moby/moby/issues/27499

RUN Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' `
  -Name ServerPriorityTimeLimit -Value 0 -Type DWord
  
# Install Url Rewrite.

WORKDIR /install
ADD https://download.microsoft.com/download/C/9/E/C9E8180D-4E51-40A6-A9BF-776990D8BCA9/rewrite_amd64.msi rewrite_amd64.msi
RUN Write-Host 'Installing URL Rewrite' ; `
    Start-Process msiexec.exe -ArgumentList '/i', 'rewrite_amd64.msi', '/quiet', '/norestart' -NoNewWindow -Wait
	
# This is not the best option but for the moment do it so the App Pool has e.g. write access rights for logs and other stuff

RUN Add-LocalGroupMember -Group 'Administrators' -Member 'IIS AppPool\DefaultAppPool';