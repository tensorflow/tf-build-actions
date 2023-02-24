# Dockerfile for the windows runner and builder.  Note unlike linux where we can easily seperate runner and image, actions runners do not yet have windows container support

FROM mcr.microsoft.com/windows/servercore:ltsc2019

# The following is needed for the actions-runner
WORKDIR /actions-runner

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop';$ProgressPreference='silentlyContinue';"]

RUN Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/v2.292.0/actions-runner-win-x64-2.292.0.zip -OutFile actions-runner-win-x64-2.292.0.zip

RUN if((Get-FileHash -Path actions-runner-win-x64-2.292.0.zip -Algorithm SHA256).Hash.ToUpper() -ne 'f27dae1413263e43f7416d719e0baf338c8d80a366fed849ecf5fffcec1e941f'.ToUpper()){ throw 'Computed checksum did not match' }

RUN Add-Type -AssemblyName System.IO.Compression.FileSystem ; [System.IO.Compression.ZipFile]::ExtractToDirectory('actions-runner-win-x64-2.292.0.zip', $PWD)

RUN Invoke-WebRequest -Uri 'https://aka.ms/install-powershell.ps1' -OutFile install-powershell.ps1; ./install-powershell.ps1 -AddToPath

RUN powershell Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

RUN powershell choco install git.install --params "'/GitAndUnixToolsOnPath'" -y

RUN powershell choco feature enable -n allowGlobalConfirmation

# Install all things needed for windows TensorFlow builds to succeed
WORKDIR /tensorflow-setup
RUN md C:\TEMP
RUN md C:\TMP
RUN powershell choco install microsoft-openjdk11
RUN powershell choco install nssm
RUN powershell choco install 7zip
RUN powershell choco install sysinternals

# TODO: copy vs config in
COPY vs2019.vsconfig .
RUN powershell choco install visualstudio2019community --config vs2019.vsconfig

# TODO: Allow python to be an arg so we can make all supported images



# The command is for the actions-runner to be able to launch as expected
CMD [ "pwsh", "-c", "./config.cmd --name $env:RUNNER_NAME --url https://github.com/$env:RUNNER_REPO --token $env:RUNNER_TOKEN --labels $env:RUNNER_LABELS --unattended --replace --ephemeral; ./run.cmd"]