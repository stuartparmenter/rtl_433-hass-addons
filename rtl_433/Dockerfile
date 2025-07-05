ARG BUILD_FROM=ghcr.io/home-assistant/amd64-base-debian:bookworm
FROM $BUILD_FROM as builder

LABEL maintainer="pavlov@pavlov.net"

ENV LANG C.UTF-8

# Install build dependencies
RUN apt-get update && apt-get install -y \
    libusb-dev \
    libusb-1.0-0-dev \
    libssl-dev \
    cmake \
    pkg-config \
    git && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /build

# --- Stage: Build librtlsdr (rtl-sdr-blog fork) ---
FROM builder as librtlsdr_builder

WORKDIR /build
RUN git clone https://github.com/rtlsdrblog/rtl-sdr-blog
WORKDIR /build/rtl-sdr-blog

# Use latest master (or pin a commit if desired)
RUN git checkout master

# Build and install to /usr with udev rules
RUN mkdir build && cd build && \
    cmake .. -DCMAKE_INSTALL_PREFIX=/usr -DINSTALL_UDEV_RULES=ON && \
    make -j4 && \
    make DESTDIR=/build/root/ install

# --- Stage: Build rtl_433 ---
FROM builder as rtl_433_builder

# Copy built librtlsdr (in /usr) into this stage
COPY --from=librtlsdr_builder /build/root/usr /usr

WORKDIR /build
RUN git clone https://github.com/merbanan/rtl_433
WORKDIR /build/rtl_433

ARG rtl433GitRevision=25.02
RUN git checkout ${rtl433GitRevision}

# Build and install to /usr
RUN mkdir build && cd build && \
    cmake .. -DCMAKE_INSTALL_PREFIX=/usr && \
    make -j4 && \
    make DESTDIR=/build/root/ install

# --- Final Stage: Runtime image ---
FROM $BUILD_FROM

ENV LANG C.UTF-8

# Install minimal runtime dependencies
RUN apt-get update && apt-get install -y \
    libusb-1.0-0 \
    sed && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /root

# Copy built librtlsdr and rtl_433 files (both installed under /usr and /etc)
COPY --from=librtlsdr_builder /build/root/ /
COPY --from=rtl_433_builder /build/root/ /

# Ensure libraries are linked
RUN ldconfig

# Add run script
COPY run.sh /
RUN chmod +x /run.sh

CMD [ "/run.sh" ]