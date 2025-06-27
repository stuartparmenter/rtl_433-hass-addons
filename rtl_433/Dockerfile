ARG BUILD_FROM=ghcr.io/home-assistant/amd64-base:3.21
FROM $BUILD_FROM as builder
MAINTAINER pavlov@pavlov.net

ENV LANG C.UTF-8

# Copied with minor edits from https://github.com/hertzg/rtl_433_docker/blob/master/images/alpine/build-context/Dockerfile
RUN apk add --no-cache --virtual .buildDeps \
    build-base \
    libusb-dev \
    libusb-1.0-0-dev \
    openssl-dev \
    cmake \
    git

WORKDIR /build

FROM builder as librtlsdr_builder

RUN git clone https://github.com/librtlsdr/librtlsdr
WORKDIR ./librtlsdr

# Build a specific commit or tag.
RUN git checkout master
WORKDIR ./build
RUN cmake ..
RUN make -j 4

WORKDIR /build/root
WORKDIR /build/librtlsdr/build
RUN make install
RUN ldconfig


FROM builder as rtl_433_builder

RUN git clone https://github.com/merbanan/rtl_433
WORKDIR ./rtl_433

# Build a specific commit or tag.
ARG rtl433GitRevision=25.02
RUN git checkout ${rtl433GitRevision}
WORKDIR ./build
RUN cmake ..
RUN make -j 4

WORKDIR /build/root
WORKDIR /build/rtl_433/build
RUN make DESTDIR=/build/root/ install

FROM $BUILD_FROM

ENV LANG C.UTF-8

RUN apk add --no-cache libusb sed
WORKDIR /root
COPY --from=rtl_433_builder /build/root/ /

# Run script
COPY run.sh /
RUN chmod a+x /run.sh

CMD [ "/run.sh" ]
