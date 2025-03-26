#
# The Angry Clanger Dockerfile
# ============================
# A linux host loaded with clang compilers for cross compiling to ARM targets. You might use
# this container as-is or as the base for a more specific docker. It is originally designed
# for cross compiling Googly/Apply projects like Flutter and Swift to ARM targets.
#
# It includes Clang v20.1.1 compilers for the following targets:
#
# - ARMv8 64bit Cross Compiler (aarch64-linux-gnu)
# - ARMv7 32bit Hard Float Cross Compiler (arm-linux-gnueabihf)
# - ARMv7 32bit Hard Float Bare Metal (arm-none-eabihf)
#
# The clang compilers are built from source and installed in the /opt/tools/clang directory
# from where they can be symlinked to the /usr/bin directory or referenced directly. The
# compilers are built using cmake and ninja-build and the binutils package is also installed.
#
# Build the Docker Image
# ======================
# docker buildx build -t clanger:20.1.1 .
# 
# Run the Docker Image
# =====================
# docker run -it clanger:20.1.1
#
FROM debian:bookworm-slim

# Defining the versions of clang and binutils to be installed up front, you 
# should be able to change these values to uprev this docker over time.
ARG VERCLANG="20.1.1"
ARG VERBINUTILS="2.43.1"

# Define the targets that we want to compile for, you can add or remove targets
# as required, don't forget you'll need to update the CMakePresets.json file to
# include the new target.
ARG TARGETS="aarch64-linux-gnu"
#ARG TARGETS="arm-none-eabihf arm-linux-gnueabihf aarch64-linux-gnu"

#
# It is preinstalled with:
# - git, curl, wget    => For fetching source code
# - cmake, ninja-build => For building source code
# - nano, python3      => For editing and running scripts
# - xz-utils           => For extracting tarballs
# - pkg-config         => For finding libraries
# - g++				   => For compiling, we do clang later
#
WORKDIR /tmp
RUN apt-get update \
	&& apt-get install -y cmake curl g++ git nano ninja-build pkg-config python3 wget xz-utils \
	&& rm -rf /var/lib/apt/lists/*

#
# Fetch the appropriate versions of the clang and binutils source code
#
RUN git clone --depth=1 --branch llvmorg-$VERCLANG https://github.com/llvm/llvm-project.git \
    && wget https://sourceware.org/pub/binutils/releases/binutils-$VERBINUTILS.tar.xz

#
# Compile Clang, currently using a cmake preset file to manage the build configuration
# but this could be simplified to provide the arguments directly here.
#
COPY CMakePresets.json /tmp/llvm-project/llvm
RUN for target in $TARGETS; do \
    echo "Compiling Clang for $target" \
        && cd /tmp/llvm-project/llvm \
        && cmake --preset="$target" \
        && cmake --build --preset="$target"; \
done

#
# Compile BinUtils, using the traditional configure, make, make install process.
#
RUN for target in $TARGETS; do \
    echo "Compiling BinUtils for $target" \
        && cd /tmp \
        && tar -xvf "binutils-$VERBINUTILS.tar.xz" \
        && cd binutils-$VERBINUTILS \
        && ./configure --prefix="/opt/tools/clang/$target" \
            --enable-gold \
            --enable-ld \
            --target="$target" \
        && make -j$(nproc) \
        && make install \
        && cd .. \
        && rm -rf binutils-$VERBINUTILS; \
done        
    
# Cleaning up to reduce the size of the docker
RUN rm -rf llvm-project \
    && rm binutils-$VERBINUTILS.tar.xz

# Set the default command (this could be a start-up script or a terminal)
CMD ["/bin/bash"]