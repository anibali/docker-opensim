FROM ubuntu:18.04 as opensim-builder

RUN apt-get update \
 && apt-get install -y \
    curl build-essential git cmake \
    openjdk-8-jdk swig \
    libgconf-2-4 liblapack-dev freeglut3-dev libxi-dev libxmu-dev doxygen \
    python3 python3-dev python3-numpy python3-setuptools \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /root

# Install Netbeans
RUN curl -sLO https://download.netbeans.org/netbeans/8.2/final/bundles/netbeans-8.2-javase-linux.sh \
 && chmod +x netbeans-8.2-javase-linux.sh \
 && ./netbeans-8.2-javase-linux.sh --silent \
 && rm netbeans-8.2-javase-linux.sh

# Download and extract OpenSim-Core.
RUN git clone https://github.com/opensim-org/opensim-core.git \
 && cd opensim-core \
 && git checkout fbf06170beb628b8f18212ac8cf3c7757d5ea7aa

# Download OpenSim-GUI.
RUN git clone https://github.com/opensim-org/opensim-gui.git \
 && cd opensim-gui \
 && git checkout 8bd2ec046905318fa863ac16beaebb033e46d6df \
 && git submodule update --init --recursive -- opensim-models opensim-visualizer Gui/opensim/threejs

# Build OpenSim-Core dependencies.
RUN mkdir build_deps \
 && cd build_deps \
 && cmake ../opensim-core/dependencies -DSUPERBUILD_ezc3d:BOOL=on -DCMAKE_INSTALL_PREFIX=~/opensim_dependencies_install \
 && cmake . -LAH \
 && cmake --build . --config Release

# Build OpenSim-Core.
RUN mkdir build_core \
 && cd build_core \
 && cmake ../opensim-core -DOPENSIM_DEPENDENCIES_DIR=~/opensim_dependencies_install -DBUILD_JAVA_WRAPPING=on -DBUILD_PYTHON_WRAPPING=on -DOPENSIM_C3D_PARSER=ezc3d -DBUILD_TESTING=off -DCMAKE_INSTALL_PREFIX=~/opensim-core-install -DOPENSIM_INSTALL_UNIX_FHS=OFF \
 && cmake . -LAH \
 && cmake --build . --config Release \
 && cmake --install . \
 && make install

# Build OpenSim-GUI.
RUN mkdir build_gui \
 && cd build_gui \
 && cmake ../opensim-gui -DCMAKE_PREFIX_PATH=~/opensim-core-install -DAnt_EXECUTABLE="/usr/local/netbeans-8.2/extide/ant/bin/ant" -DANT_ARGS="-Dnbplatform.default.netbeans.dest.dir=/usr/local/netbeans-8.2;-Dnbplatform.default.harness.dir=/usr/local/netbeans-8.2/harness" \
 && make CopyOpenSimCore \
 && make CopyVisualizer \
 && make CopyModels \
 && make PrepareInstaller

################################################################################

FROM ubuntu:18.04

# Create a non-root user. All users can use /home/user as their home directory.
RUN adduser --disabled-password --gecos '' --shell /bin/bash user \
 && chmod 777 /home/user
ENV HOME=/home/user
WORKDIR /home/user

# Install OpenSim dependencies.
RUN apt-get update \
 && apt-get install -y \
    openjdk-8-jre \
    liblapack3 \
    libgconf-2-4 \
 && rm -rf /var/lib/apt/lists/*

# Prepare OpenSim install folder.
RUN mkdir -p /opt/opensim \
 && chown -R user:user /opt/opensim

# Switch to the non-root user.
USER user

# Install OpenSim.
COPY --chown=user:user --from=opensim-builder /root/opensim-gui/Gui/opensim/dist/OpenSim-4.2-2020-09-02-8bd2ec04.tar.gz /home/user/
RUN tar xzf OpenSim-4.2-2020-09-02-8bd2ec04.tar.gz \
 && rm OpenSim-4.2-2020-09-02-8bd2ec04.tar.gz \
 && cd OpenSim \
 && cp -R * /opt/opensim \
 && cd /opt/opensim/bin \
 && rm -rf /home/user/OpenSim

# Set up environment variables.
ENV OPENSIM_HOME=/opt/opensim \
    LIBRARY_PATH=/opt/opensim/lib:$LIBRARY_PATH \
    LD_LIBRARY_PATH=/opt/opensim/lib:$LD_LIBRARY_PATH \
    PATH=/opt/opensim/bin:$PATH

CMD [ "/opt/opensim/bin/OpenSim" ]
