# WSL and VSCode SSH Setup Guide for connecting to vast.ai (Windows 10)

## Install and enable WSL


## Install VSCode and extensions
Install **Remote Development** extension.

# Set up SSH for connecting to vast.ai
## 1. Checking for existing SSH keys
Run this script in WSL terminal:
```bash
ls -al ~/.ssh
```
If you see files like `id_rsa` or `id_ed25519`, you may already have a key. You can reuse it (ignore step 2) or create a new one.


## 2. Generate new SSH key
Run thí script in WSL terminal:
```bash
ssh-keygen -t ed25519 -C "youremail"
```


## 3. Add public SSH Key to SSH agent
Checking SSH agent:
```bash 
eval "$(ssh-agent -s)"
```
Then add SSH Key to SSH Agent:
```bash
ssh-add ~/.ssh/id_ed25519
```


## 4. Add public SSH key to vast.ai
Run this script in WSL to read the SSH private key:
```bash
cat ~/.ssh/id_ed25519.pub
```
Go to https://cloud.vast.ai/manage-keys/ and then paste the key into the **SSH Keys** field.


## 5. Setting WSL SSH in VSCode
Create and save `ssh.bat` contain the following script: 
```bash
@echo off
setlocal

set v_params=%*
set v_params=%v_params:\\wsl.localhost\Ubuntu\home\<YourUserName>=/home/<YourUserName>%
set v_params=%v_params:\=/% 
set v_params=%v_params:c:=/mnt/c%

C:\Windows\System32\wsl.exe ssh %v_params%

endlocal
```
Open '*User Settings JSON*' in VSCode and add the following script:
```json
"remote.SSH.path": "folder_path_contain_ssh_bat\\ssh.bat",
"remote.SSH.showLoginTerminal": true,
"remote.SSH.configFile": "\\\\wsl.localhost\\Ubuntu\\home\\<YourUserName>\\.ssh\\config",
"security.allowedUNCHosts": [
    "wsl$",
    "wsl.localhost"
],
```

## 6. Add vast.ai SSH conection config
1. Go to https://cloud.vast.ai/create/ and rent a GPU.
2. Click the '*Open terminal access*' button at the middle bottom GPU.
3. At '*Direct ssh connect*' field you may see something like `ssh -p 51729 root@66.115.179.150 -L 8080:localhost:8080` which the `51729` is the port, `root` is the user, `66.115.179.150` is the HostName.
4. Open **VSCode** (from *Start Menu* or *Explorer*).
5. `ctrl + shift + P`, type '*Remote-SSH: Open SSH Configuration file*'and choose '*\\\wsl.localhost\Ubuntu\home\<YourUserName>\.ssh\config*'.
6. Add the bellow script:
```
Host vastai
    HostName 66.115.179.150
    User root
    Port 51729
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
```


## 7. Setup vast.ai GPU server
Open WSL terminal, and copy SSH private key:
```bash
cat ~/.ssh/id_ed25519
```
Create file *vastai_setup.sh*:
```bash
# ======= Update lastest Ubuntu packages =======
apt-get update
apt-get install -y curl git-core

# ======= Install and enable Git LFS =======
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash 
sudo apt-get install git-lfs -y
git lfs install # Verify the installation was successful

# ======= Set up SSH key to connect github =======
mkdir -p ~/.ssh && chmod 700 ~/.ssh
cat > ~/.ssh/id_ed25519 <<- EOM
-----BEGIN OPENSSH PRIVATE KEY-----
<paste your copied SSH private key>
-----END OPENSSH PRIVATE KEY-----
EOM
chmod 400 ~/.ssh/id_ed25519

ssh-keygen -F github.com || ssh-keyscan github.com >> ~/.ssh/known_hosts

git config --global user.email "youremail"
git config --global user.name "yourname"

# ======= Install Miniconda (if not already installed) =======
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh
bash miniconda.sh -b -p /opt/conda
export PATH="/opt/conda/bin:$PATH"
source /opt/conda/etc/profile.d/conda.sh
```
In WSL terminal, copy *vastai_setup.sh* to vast.ai Server by:
```bash
scp -r /mnt/<FolderPath>/vastai_setup.sh vastai:/workspace
```
Connect vast.ai via SSH: 
```bash
ssh vastai
```
Run setup file: 
```bash
bash vastai_setup.sh
```
