FROM python:3-alpine AS dev

ARG USER_ID=1000
ARG GROUP_ID=1000
ARG USERNAME=kainga
ARG USER_HOME=/home/${USERNAME}

WORKDIR /app

COPY ./start-container /usr/local/bin/start-container
RUN chmod +x /usr/local/bin/start-container

#Allow user to fail fast and safe
RUN set -ex && \ 

    # We use the passwd database so as to query the username, UID, primary GID, home directory, shell
    if getent passwd ${USER_ID}; then \    
        EXISTING_USER_NAME=$(getent passwd ${USER_ID} | cut -d: -f1); \
        deluser ${EXISTING_USER_NAME}; \
    fi && \

    # We use the group database so as to query the group name, GID, member list
    if getent group ${GROUP_ID}; then \
        EXISTING_GROUP_NAME=$(getent group ${GROUP_ID} | cut -d: -f1); \
        delgroup ${EXISTING_GROUP_NAME}; \
    fi && \
    \
    addgroup -S -g ${GROUP_ID} ${USERNAME} && \
    adduser -S -D -u ${USER_ID} -G ${USERNAME} -h ${USER_HOME} ${USERNAME} && \
    \
    mkdir -p ${USER_HOME} && \
    chown -R ${USER_ID}:${GROUP_ID} ${USER_HOME}

# -------------------------------------------------------------------------------
# Set PATH to include the user's local Python binary directory (~/.local/bin)
# -------------------------------------------------------------------------------
# When installing Python packages using pip without root privileges inside the 
# container, binaries (like `virtualenv`) are placed in the user's local binary 
# directory, typically at `/home/<username>/.local/bin`. By default, this directory 
# is not included in the PATH, which means commands installed there cannot be 
# executed directly from the shell.
#
# In this Dockerfile, we create a non-root user (`kainga`) to run development 
# tasks securely. Pip will therefore default to installing packages in the user's 
# home directory, not system-wide. To ensure the container can execute binaries 
# like `virtualenv` without specifying the full path, we explicitly prepend 
# the user's local binary directory to the PATH environment variable.
#
# This step ensures:
#  1. All scripts and commands installed by pip in user mode are immediately 
#     available in the shell.
#  2. The container remains secure by running as a non-root user.
#  3. No confusion or errors occur when running commands like `virtualenv` or any 
#     other pip-installed executable that would otherwise be "not found".
#
# Note: Using a system-wide installation of virtualenv is an alternative, but 
# adding the user's local bin directory to PATH maintains consistency with a 
# development workflow where non-root package installs are preferred.
ENV PATH=$USER_HOME/.local/bin:$PATH

USER ${USERNAME}

# We do docker for "dummies" XD to keep the container running tailing dev null
ENTRYPOINT ["/usr/local/bin/start-container"]
