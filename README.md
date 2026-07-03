# Container-in-Container Sandbox

Ever needed to run something in a container because you didn't trust it? Me too.
Ever run something in a container but then needed it to have the ability to also create and use containers? Me too.
Ever then get very frustrated because docker-in-docker just doesn't work, and you then have to spin up a virtual machine or a sacrificial YOLO box? Or just bail on the idea? Me too.
That's why I'm glad to share this.

Here it is in other terms if you need "LinkedIn speak"
A lightweight, high-performance sandbox blueprint designed for executing untrusted AI agents, running unverified code, or validating raw docker builds natively without risking host security or risking Docker-in-Docker filesystem collisions.

This solution uses podman + fuse-overlayfs + iptables-nft to let you nest podman containers setup as a .devcontainer.
This is what I use as a prototype when making setups for testing untrusted code or when I need to run agents in consulting jobs.

If you like this, maybe you can get someone to hire me, or drop me a tip.  Things are grim these days in the job market and the landord cometh. my ko-fi is joncaudill [link](https://ko-fi.com/joncaudill "Ko-Fi Link").

## 1. Host Machine Requirements
For Arch based systems:
Ensure your physical host machine has the core unprivileged routing and layering utilities active before initializing:
```bash
sudo pacman -S podman fuse-overlayfs iptables-nft
```
For Debian-based systems:
Install the core runtime utilities and explicitly switch your system to use the modern Netfilter/nftables network backend:
```bash
# 1. Install the core container, filesystem, and network utilities
sudo apt-get update && sudo apt-get install -y \
    podman \
    fuse-overlayfs \
    iptables

# 2. Force the host system to use the modern nftables backend 
# (Critical for nested container network bridging)
sudo update-alternatives --set iptables /usr/sbin/iptables-nft
```
## 1.5 VS Code Host Configuration 
Before opening the workspace, you must tell the VS Code Dev Containers extension to use the Podman binary instead of the default Docker daemon on your host machine.

1. Open VS Code on your Arch host.
2. Press `Ctrl + Shift + P` to open the command palette.
3. Select **Preferences: Open User Settings (JSON)**.
4. Add these two lines inside your main settings JSON block:

```json
{
    "dev.containers.dockerPath": "podman",
    "dev.containers.dockerComposePath": "podman-compose"
}
```
## 1.7 Alternative IDE Host Configuration
If you are not using VS Code but use another major IDE or tool that supports the open-source devcontainer spec, you must point its container executor to Podman.

### Option A: JetBrains IDEs
JetBrains handles devcontainers natively through its central Docker plugin ecosystem.
1. Go to **Settings** -> **Build, Execution, Deployment** -> **Docker**.
2. Click the **`+`** icon to add a new connection.
3. Change the connection type from Docker to **Podman**.
4. Under **Settings** -> **Build, Execution, Deployment** -> **Dev Containers**, ensure the default Docker environment drops down to use your newly added Podman connection profile.

### Option B: The Official Devcontainer CLI Tool
If you are using a lightweight terminal setup, Neovim, or a platform without GUI extension settings, you can launch this entire blueprint using the official, open-source Node-based Devcontainer CLI tool.
1. Install the CLI utility globally on your host:
   ```bash
   sudo npm install -g @devcontainers/cli
   ```
2. Launch the sandbox directly from your project directory by overriding the target path flag:
   ```bash
   devcontainer up --workspace-folder . --docker-path podman
   ```

---

## 2. Devcontainer Sandbox Engine Layout
The files, as-is, in the `.devcontainer` directory will set up what is necessary for the nestable podman environment with a basic alpine linux install.

From inside the devcontainer, you can also setup nested podman instances.
An alias has been set up for `docker` inside the container so that any agents expecting to use docker will be able to do so.

- `.devcontainer/Dockerfile`: Configures the basic environment. Installs the required drivers and tools to make it work, and to get around the locking errors.
- `.devcontainer/devcontainer.json`: Tells your IDE to map your project directory, keep the container awake, and to allocate system capabilities to support the child runtimes (i.e. `--privileged`)

In the `Dockerfile` as noted in the comments, anything above that is just like any other docker file.  You can install things and run things, and use whatever you need to use in addition to containers.  Just don't delete the `podman`, `iptables` and `fuse-overlayfs` install, or it won't work.

Anything else, you can leave alone.  If you mess with it and it breaks...that's why.

To start off, copy the .devcontainer directory into your project directory, (or if you know what you're doing, the parts you need into an existing .devcontainer directory), and add anything that you need to your dockerfile that you might need in the container (languages, tools, etc.)

## 3. Deployment Playbook
1. Open the project directory in your IDE. If it prompts to reopen as a devcontainer and you accept, you can skip to step 4.
2. Press `Ctrl + Shift + P` to open the command palette.
3. Select **`Dev Containers: Reopen in Container`**.
4. Run standard commands natively inside the terminal that you need for your project.  Everything you create in the container in the /workspace directory will still exist and be there for you when you exit the container.
