# Setting the parent image to ubuntu
FROM ubuntu:18.04
LABEL vendor="The Batfish Open Source Project"

ARG GOOGLE_JAVA_FORMAT_VERSION=1.7
ARG JACOCO_VERSION=0.8.2
ARG MAVEN_VERSION=3.6.3
ARG BAZELISK_VERSION=1.3.0
ARG USERNAME=batfish
ARG UID=2000

# Set a Working Dir
User root
WORKDIR /root/workdir

# Installing Java, python, z3 dependencies
RUN apt-get update && apt-get install -y \
    apt-transport-https \
    binutils \
    ca-certificates \
    curl \
    gcc \
    git \
    jq \
    lsb-release \
    net-tools \
    openjdk-8-jdk \
    python3-dev \
    rsync \
    software-properties-common \
    unzip \
    wget \
    zip \
&&  rm -rf /var/lib/apt/lists/* \
&&  apt-get clean

# Install pip and virtualenv
RUN curl https://bootstrap.pypa.io/get-pip.py | python3 \
&& python3 -m pip install 'virtualenv==20.0.5'

# Install Z3
RUN wget --no-verbose https://raw.githubusercontent.com/batfish/batfish/master/tools/install_z3.sh \
&& bash install_z3.sh \
&& rm -r ~/.batfish_z3_cache \
&& rm install_z3.sh

# Install bazelisk
RUN wget https://github.com/bazelbuild/bazelisk/releases/download/v${BAZELISK_VERSION}/bazelisk-linux-amd64 \
&&  install -m0755 bazelisk-linux-amd64 /usr/local/bin/bazelisk \
&&  ln -s /usr/local/bin/bazelisk /usr/local/bin/bazel \
&&  rm bazelisk-linux-amd64

##### Switch to non-root-user

RUN groupadd -g ${UID} ${USERNAME} \
&& useradd -m -r -u ${UID} -g ${USERNAME} ${USERNAME} \
&& mkdir -p /home/${USERNAME}/workdir \
&& chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}
USER ${USERNAME}
WORKDIR /home/${USERNAME}/workdir

# Create AWS-CLI virtualenv
RUN python3 -m virtualenv .venv-aws && . .venv-aws/bin/activate && python3 -m pip install 'awscli==1.18.5' && deactivate

# Install maven
RUN wget --no-verbose https://www-us.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.zip \
&& unzip apache-maven-${MAVEN_VERSION}-bin.zip \
&& rm -f apache-maven-${MAVEN_VERSION}-bin.zip

# Setup JAVA_HOME
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64/
# Set Java to obey cgroup limits
ENV _JAVA_OPTIONS -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1

# Add Maven to PATH
ENV PATH $PATH:/home/${USERNAME}/workdir/apache-maven-${MAVEN_VERSION}/bin:/home/${USERNAME}/bin

# Fetch all the current Maven dependencies and bazel version
RUN git clone --depth=1 --branch=master https://github.com/batfish/batfish \
&& cd batfish \
&& mvn -f projects verify -DskipTests=false \
&& mvn -f projects dependency:get -Dartifact=com.google.googlejavaformat:google-java-format:${GOOGLE_JAVA_FORMAT_VERSION}:jar:all-deps \
&& mvn -f projects dependency:get -Dartifact=org.jacoco:org.jacoco.cli:${JACOCO_VERSION}:jar:nodeps \
&& bazel version \
&& cd .. \
&& rm -rf batfish
