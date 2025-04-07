# Base image
FROM osrf/ros:jazzy-desktop

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV ROS_WS=/ros2_ws

# Update system and install tools
RUN apt-get update && apt-get install -y \
    build-essential \
    git \
    wget \
    curl \
    python3-pip \
    nginx \
    ros-${ROS_DISTRO}-rosbridge-suite \
    && rm -rf /var/lib/apt/lists/*

# Install cpp-httplib (header-only)
RUN mkdir -p /opt/httplib && \
    curl -L https://raw.githubusercontent.com/yhirose/cpp-httplib/master/httplib.h -o /opt/httplib/httplib.h

ENV CPATH="/opt/httplib:$CPATH"

# Create a non-root user with fallback UID
ARG USERNAME=rosuser
ARG USER_UID=1000
ARG USER_GID=$USER_UID
ARG FALLBACK_UID=1001

RUN set -e && \
    # Determine group name to use
    if getent group ${USER_GID} > /dev/null 2>&1; then \
        USER_GID_NAME=$(getent group ${USER_GID} | cut -d: -f1); \
    else \
        groupadd -g ${USER_GID} ${USERNAME}; \
        USER_GID_NAME=${USERNAME}; \
    fi && \
    # Try to create user with UID 1000, if it fails try with fallback UID 1001
    if ! id -u ${USER_UID} > /dev/null 2>&1; then \
        useradd -u ${USER_UID} -g ${USER_GID_NAME} -m ${USERNAME}; \
        usermod -aG sudo ${USERNAME}; \
    elif ! id -u ${FALLBACK_UID} > /dev/null 2>&1; then \
        # Fallback to UID 1001 if UID 1000 is already in use
        useradd -u ${FALLBACK_UID} -g ${USER_GID_NAME} -m ${USERNAME}; \
        usermod -aG sudo ${USERNAME}; \
        USER_UID=${FALLBACK_UID}; \
    fi && \
    # Ensure /ros2_ws exists
    mkdir -p /ros2_ws && \
    # Make sure user exists before chown
    if id -u ${USERNAME} > /dev/null 2>&1; then \
        chown -R ${USERNAME}:${USER_GID_NAME} /ros2_ws /opt/httplib; \
    fi

# Switch to the new user
USER $USERNAME

# Create workspace
RUN mkdir -p ${ROS_WS}/src
WORKDIR ${ROS_WS}

# Init colcon workspace
# RUN /bin/bash -c "source /opt/ros/${ROS_DISTRO}/setup.bash && colcon build"

# Expose ports:
# 8080 - for serving web interface (cpp-httplib or nginx)
# 9090 - for rosbridge websocket
EXPOSE 8080 9090

# Optionally start nginx (edit if needed)
# COPY default.conf /etc/nginx/sites-available/default

# Start point: ROS2 + optionally nginx/your app
CMD ["/bin/bash"]
