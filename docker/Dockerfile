# Use the osrf/ros:humble-desktop image as the base for the ci stage
ARG ROS_DISTRO=humble
FROM osrf/ros:${ROS_DISTRO}-desktop AS ci

ENV DEBIAN_FRONTEND=noninteractive

WORKDIR /root/blue_ws
COPY . src/blue

# Install additional apt packages needed for CI
RUN apt-get -q update \
    && apt-get -q -y upgrade \
    && apt-get -q install --no-install-recommends -y \
        git \
        sudo \
        clang \
        clang-format-14 \
        clang-tidy \
        clang-tools \
        python3-pip \
        python3-dev \
        software-properties-common \
        nano \
        iputils-ping \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*


RUN python3 -m pip install opencv-python

RUN apt-get -q update \
    && apt-get -q -y upgrade \
    && rosdep update \
    && rosdep install -y --from-paths src --ignore-src --rosdistro ${ROS_DISTRO} --as-root=apt:false --skip-keys='opencv-python' \
    && rm -rf src \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

FROM ci AS robot

# Configure a new non-root user
ARG USERNAME=blue
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    && usermod -a -G dialout $USERNAME \
    && echo "source /usr/share/bash-completion/completions/git" >> /home/$USERNAME/.bashrc

ENV DEBIAN_FRONTEND=noninteractive

USER $USERNAME
ENV USER=$USERNAME

# Install some core apt packages as non-root user
RUN sudo apt-get -q update \
    && sudo apt-get -q -y upgrade \
    && sudo apt-get -q install --no-install-recommends -y \
        lsb-release \
        wget \
        gnupg \
    && sudo apt-get autoremove -y \
    && sudo apt-get clean -y \
    && sudo rm -rf /var/lib/apt/lists/*

ENV USER_WORKSPACE=/home/$USERNAME/blue_ws
WORKDIR $USER_WORKSPACE
COPY --chown=$USER_UID:$USER_GID . src/blue

# Additional Python packages
RUN python3 -m pip install transforms3d pymavlink

# Again, skip opencv-python
RUN sudo apt-get -q update \
    && sudo apt-get -q -y upgrade \
    && rosdep update \
    && rosdep install -y --from-paths src --ignore-src --rosdistro ${ROS_DISTRO} --skip-keys='opencv-python' \
    && sudo apt-get autoremove -y \
    && sudo apt-get clean -y \
    && sudo rm -rf /var/lib/apt/lists/*

# Setup the environment variables
#RUN echo "source ${USER_WORKSPACE}/install/setup.bash" >> /home/$USERNAME/.bashrc \
#    && echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> /home/$USERNAME/.bashrc \
#    && echo "export RMW_IMPLEMENTATION=rmw_zenoh_cpp" >> /home/$USERNAME/.bashrc

FROM robot AS desktop

ENV DEBIAN_FRONTEND=noninteractive

WORKDIR /home/$USERNAME/blue_ws/src

WORKDIR /home/$USERNAME/blue_ws

RUN sudo apt-get autoremove -y \
    && sudo apt-get clean -y \
    && sudo rm -rf /var/lib/apt/lists/*

RUN echo "colcon build..........................."

# It's better to use a shell to source the setup script and build
RUN /bin/bash -c "source /opt/ros/${ROS_DISTRO}/setup.sh && colcon build"

USER root
RUN apt-get update && apt-get install -y \
    ros-${ROS_DISTRO}-joy \
    && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y \
    ros-${ROS_DISTRO}-joy-linux \
    && rm -rf /var/lib/apt/lists/*

RUN python3 -m pip install transforms3d pymavlink transformations

RUN apt-get update && apt-get install -y \
    ros-${ROS_DISTRO}-rviz2 \
    libboost-filesystem-dev libboost-system-dev liboctomap-dev \
    && rm -rf /var/lib/apt/lists/*

# Install scipy
RUN python3 -m pip install scipy

COPY new_rov/ros-humble-rmw-zenoh-cpp_0.1.1-1jammy.20250325.181115_amd64.deb /ros-humble-rmw-zenoh-cpp_0.1.1-1jammy.20250325.181115_amd64
COPY new_rov/ros-humble-zenoh-cpp-vendor_0.1.1-1jammy.20250207.195634_amd64.deb /ros-humble-zenoh-cpp-vendor_0.1.1-1jammy.20250207.195634_amd64.deb
RUN dpkg -i /ros-humble-zenoh-cpp-vendor_0.1.1-1jammy.20250207.195634_amd64.deb && dpkg -i /ros-humble-rmw-zenoh-cpp_0.1.1-1jammy.20250325.181115_amd64 
RUN echo "export RMW_IMPLEMENTATION=rmw_zenoh_cpp" >> /etc/bash.bashrc
ENV RMW_IMPLEMENTATION=rmw_zenoh_cpp

RUN echo "source ${USER_WORKSPACE}/install/setup.bash" >> /home/$USERNAME/.bashrc \
    && echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> /home/$USERNAME/.bashrc \
    && echo "export RMW_IMPLEMENTATION=rmw_zenoh_cpp" >> /home/$USERNAME/.bashrc

USER $USERNAME
