FROM library/python:3.12-slim-trixie
# install needed packages
RUN DEBIAN_FRONTEND=noninteractive apt-get update && \
    apt-get install -y sudo sshpass curl wget git openssh-client libhdf5-dev libssl-dev libffi-dev nano nmap bind9-dnsutils && \
    apt-get clean all && rm -rf /var/lib/apt/lists/*
# base container setup
ARG USERNAME=vscode \
    USER_UID=1000 \
    USER_GID=1000
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
    && chmod 0440 /etc/sudoers.d/$USERNAME
# install ansible things
COPY requirements-base.txt .
RUN pip3 install --upgrade -r requirements-base.txt && \
    rm -rf /root/.cache/pip && \
    mkdir -p /etc/ansible && \
    echo 'localhost' > /etc/ansible/hosts
# install extra packages
COPY requirements.txt .
RUN pip3 install --upgrade -r requirements.txt && \
    rm -rf /root/.cache/pip
# install collections
COPY requirements.yml .
USER $USERNAME
RUN ansible-galaxy install -r requirements.yml
ENTRYPOINT ["/bin/bash"]
