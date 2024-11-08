# STM32 Development Container

See [official specs](https://containers.dev/) and [vscode docs](https://code.visualstudio.com/docs/devcontainers/containers) for more information about dev containers and how to use.

The STM32 Audit container has been designed to run as both a standalone container and a dev container primarily to cater to users who either want to use their own setup for auditing (e.g. Vim/Emacs) or use VS Code.

As the container provides the whole audit environment, no attempt is made to reduce image sizes.

Before using this, you will need to download official packages from the ST website and place in the [third_party](third_party/) folder for each of:

* STM32L4
* XCUBEIDE
* SBSFU
* STSAFE

It's likely you will need to update the filename args used to build the container if the defaults no longer work.
