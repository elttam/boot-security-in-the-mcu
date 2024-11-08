FROM ubuntu:22.04

ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

ARG SBSFU_ZIP=./third_party/SBSFU/en.stm32cube-expansion-sbsfu-v2.6.2.zip
ARG XCUBEIDE_ZIP=./third_party/XCUBEIDE/en.st-stm32cubeide_1.15.1_21094_20240412_1041_amd64.deb_bundle.sh.zip
ARG XCUBEIDE_INSTALLER=st-stm32cubeide_1.15.1_21094_20240412_1041_amd64.deb_bundle.sh
ARG STM32CUBEL4_ZIP=./third_party/STM32L4/en.stm32cubel4-v1-18-1.zip

COPY ${SBSFU_ZIP} /opt/SBSFU/
COPY ${XCUBEIDE_ZIP}  /opt/XCUBEIDE/
COPY ${STM32CUBEL4_ZIP} /opt/STM32L4/

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
      sudo \
      unzip \
      xchm
  apt-get autoremove -y
  apt-get clean -y
  rm -rf /var/lib/apt/lists/*
EOF

# dependencies of sbsfu prebuild.sh
RUN <<EOF
  export DEBIAN_FRONTEND=noninteractive
  apt-get update
  apt-get -y install --no-install-recommends \
      python-is-python3 \
      python3 \
      python3-ecdsa \
      python3-numpy \
      python3-pip \
      python3-pycryptodome \
      python3-pyelftools
  apt-get autoremove -y
  apt-get clean -y
  rm -rf /var/lib/apt/lists/*
EOF

# # install stm32cubeide
RUN <<EOF
  cd /opt/XCUBEIDE
  apt-get update
  unzip $(basename ${XCUBEIDE_ZIP})
  yes | LICENSE_ALREADY_ACCEPTED=1 sh ${XCUBEIDE_INSTALLER}
  apt-get autoremove -y
  apt-get clean -y
  rm -rf /var/lib/apt/lists/*
EOF

# # wow.. just wow.. how can the permissions be so wrong by default!?!? stm32cubeide will fail on builds
# # try and fix that by ensuring shell scripts are executable.
RUN <<EOF
  cd /opt/SBSFU
  unzip $(basename ${SBSFU_ZIP})
  find /opt/SBSFU/ -type f -iname '*\.sh' -execdir chmod u+x {} \;
EOF

# create non-root user account with sudo and ownership over /opt
RUN <<EOF
  group_name="${USERNAME}"
  groupadd --gid ${USER_GID} ${USERNAME}
  useradd -s /bin/bash --uid ${USER_UID} --gid ${USER_GID} -m ${USERNAME}

  if [ "${USERNAME}" != "root" ]; then
      echo ${USERNAME} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${USERNAME}
      chmod 0440 /etc/sudoers.d/$USERNAME
  fi

  chown -R ${USERNAME}:${group_name} /opt
EOF

USER ${USERNAME}
