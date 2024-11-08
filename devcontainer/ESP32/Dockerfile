ARG IDF_VERSION=release-v5.3
ARG IDF_INSTALL_TARGETS=all

# espressif image is based on Ubuntu 22.04/jammy.
FROM espressif/idf:${IDF_VERSION}

ARG ESP32_TARGET_ARCH=esp32
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# install useful tooling
RUN <<EOF
  apt-get update
  export DEBIAN_FRONTEND=noninteractive
  yes | unminimize 2>&1
  apt-get -y install --no-install-recommends \
      man-db \
      manpages \
      manpages-dev \
      sudo
  apt-get autoremove -y
  apt-get clean -y
  rm -rf /var/lib/apt/lists/*
EOF

# create non-root user account with sudo access and ownership over /opt
RUN <<EOF
  # Create or update a non-root user to match UID/GID.
  group_name="${USERNAME}"
  groupadd --gid ${USER_GID} ${USERNAME}
  useradd -s /bin/bash --uid ${USER_UID} --gid ${USER_GID} --groups dialout -m ${USERNAME}

  # Add sudo support for non-root user
  if [ "${USERNAME}" != "root" ]; then
      echo ${USERNAME} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${USERNAME}
      chmod 0440 /etc/sudoers.d/$USERNAME
  fi

  chown -R ${USERNAME}:${group_name} /opt
EOF

USER ${USERNAME}

# output compile_commands.json when building projects
ENV CMAKE_EXPORT_COMPILE_COMMANDS=1

# ensure clangd is available
RUN <<EOF
  sudo update-ca-certificates --fresh
  ${IDF_PATH}/tools/idf_tools.py --non-interactive install esp-clang
  rm -rf $IDF_TOOLS_PATH/dist
EOF

RUN cp -r ${IDF_PATH}/examples/get-started/hello_world /opt
COPY --chown=${USERNAME} files/sdkconfig files/partitions.csv files/secure_boot_signing_key.pem /opt/hello_world/
COPY --chown=${USERNAME} files/clangd.config ${IDF_PATH}/components/.clangd

RUN <<EOF
  source ${IDF_PATH}/export.sh
  cd /opt/hello_world
  idf.py build
  cp build/bootloader/compile_commands.json /opt/esp/idf/components/
EOF

ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8

# load esp-idf into the env when opening a new shell
# there is a get_idf alias one can use in bash to source export.sh.
# it is also sourced by default in entrypoint.sh (ENTRYPOINT). 
RUN echo "source ${IDF_PATH}/export.sh" >> ~/.bashrc