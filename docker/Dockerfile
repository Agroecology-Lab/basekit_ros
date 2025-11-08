FROM ros:humble

# Force HTTPS for Ubuntu mirrors
RUN sed -i 's|http://archive.ubuntu.com/ubuntu/|https://archive.ubuntu.com/ubuntu/|g' /etc/apt/sources.list

# Install required utilities and update CA certificates
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    gnupg2 \
    lsb-release \
    wget && \
    update-ca-certificates

# Remove duplicate ROS 2 sources to prevent warnings
RUN rm -f /etc/apt/sources.list.d/ros2.sources

# Add ROS 2 Humble repository explicitly with HTTPS
RUN sh -c 'echo "deb [arch=$(dpkg --print-architecture)] https://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" > /etc/apt/sources.list.d/ros2.list'

# Optional: install libvdpau1 first to avoid dependency issues
RUN apt-get update && apt-get install -y libvdpau1 || apt-get -f install -y

# Install ROS 2 packages and utilities
RUN apt-get update && apt-get install -y --no-install-recommends \
    -o Acquire::https::Verify-Peer=false \
    -o Acquire::https::Verify-Host=false \
    ros-humble-domain-bridge \
    ros-humble-rmw-cyclonedds-cpp \
    ros-humble-foxglove-bridge \
    ros-humble-xacro \
    ros-humble-rosbag2-storage-mcap \
    ros-humble-foxglove-msgs \
    ros-humble-tf-transformations \
    ros-humble-septentrio-gnss-driver \
    ros-humble-usb-cam \
    ros-humble-image-transport \
    ros-humble-image-transport-plugins \
    ros-humble-image-tools \
    ros-humble-compressed-image-transport \
    ros-humble-axis-camera \
    python3-pip \
    python3-serial \
    python3-requests \
    python3-yaml \
    git \
    nano \
    unzip \
    iputils-ping


# Setup lizard
RUN mkdir -p /root/.lizard && \
    cd /root && \
    LATEST_VERSION=$(curl -s https://api.github.com/repos/zauberzeug/lizard/releases/latest | grep -oP '"tag_name": "\K(.*)(?=")') && \
    wget https://github.com/zauberzeug/lizard/releases/download/${LATEST_VERSION}/lizard_firmware_and_devtools_${LATEST_VERSION}_esp32.zip && \
    unzip lizard_firmware_and_devtools_${LATEST_VERSION}_esp32.zip -d /root/.lizard && \
    chmod +x /root/.lizard/espresso.py && \
    cd /root/.lizard && \
    pip install -r /root/.lizard/requirements.txt && \
    cd /root && \
    rm -f lizard_firmware_and_devtools_${LATEST_VERSION}_esp32.zip


# Download requirements-dev.txt from GitHub
RUN wget https://raw.githubusercontent.com/zauberzeug/basekit_ros/refs/heads/main/requirements.txt -O /root/requirements.txt && \
    wget https://raw.githubusercontent.com/zauberzeug/basekit_ros/refs/heads/main/requirements-dev.txt -O /root/requirements-dev.txt

# Install dev requirements
RUN pip install -r /root/requirements-dev.txt

COPY ./docker/bashrc /tmp/bashrc
RUN cat /tmp/bashrc >> /root/.bashrc && rm -f /tmp/bashrc

# Copy ROS packages
COPY ./basekit_driver /workspace/src/basekit_driver
COPY ./basekit_launch /workspace/src/basekit_launch
COPY ./basekit_ui /workspace/src/basekit_ui

# Build ROS packages
WORKDIR /workspace
RUN . /opt/ros/humble/setup.sh && \
    colcon build --packages-select basekit_driver basekit_launch basekit_ui --symlink-install && \
    . install/setup.sh

WORKDIR /workspace
