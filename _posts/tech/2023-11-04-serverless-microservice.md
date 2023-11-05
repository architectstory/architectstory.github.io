---
sidebar:
  nav: "tech"
---

# Serverless Microservice

## WSL 설치 (Windows Subsystem for Linux)
참고 : https://www.44bits.io/ko/post/wsl2-install-and-basic-usage
1. windows powershell 관리자 권한 실행, 아래의 명령어 수행
2. dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
3. dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
4. windows 재부팅
5. windows powershell 관리자 권한 실행, 아래의 명령어 수행
6. wsl --install
7. wsl --update (or wsl2 linux kernel update https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
8. powershell 관리자 권한 실행, 아래의 명령어 수행
9. wsl --list --online 
10. wsl --install -d Ubuntu
11. id / password 입력 (hoony / XXXX)
12. 설치 종료 후 exit
12. wsl -l -v (version 1인 경우 2로 변경 wsl --set-version Ubuntu 2)
13. wsl -t Ubuntu (강제 종료)
 - wsl 파일 위치
   C:\Users\home\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_79rhkp1fndgsc\LocalState
14. wsl

## PODMAN 설치
https://podman-desktop.io/downloads/windows
podman machine init
podman machine start
dapr init --container-runtime podman
podman machine set --rootful
podman machine stop