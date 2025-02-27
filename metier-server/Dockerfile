FROM fedora:35 AS base

FROM base AS git
RUN dnf install -y git

FROM git AS download
ARG METIER_REPO="https://github.com/Open-Transactions/metier-server"
ARG METIER_COMMIT="285578b1a28b592bed470b3bff82b3cc3a13ee92"
RUN mkdir -p /usr/src && git clone --recursive "${METIER_REPO}" /usr/src/metier-server && cd /usr/src/metier-server && git reset --hard "${METIER_COMMIT}"

FROM opentransactions/fedora-devel:29 AS compile
RUN dnf install -y \
    gcc-c++ \
    cmake \
    ninja-build \
    boost-devel \
    protobuf-compiler \
    lmdb-devel \
    protobuf-lite-devel \
    libsodium-devel \
    openssl-devel \
    zeromq-devel \
    boost-devel \
    zlib-devel

COPY --from=download /usr/src/metier-server /usr/src/metier-server
ARG UNITY_BATCH=0
RUN mkdir -p /tmp/metier-server && cd /tmp/metier-server \
    && cmake -GNinja -DCMAKE_BUILD_TYPE=Release /usr/src/metier-server \
       -DCMAKE_UNITY_BUILD=ON -DCMAKE_UNITY_BUILD_BATCH_SIZE="${UNITY_BATCH}" \
    && cmake --build . \
    && cmake --install . \
    && rm -rf /tmp/metier-server*

FROM opentransactions/fedora:29 AS run
COPY --from=compile /usr/local/bin/metier-server /usr/bin/metier-server
ENTRYPOINT [ "/usr/bin/metier-server", "--data_dir=/metier" ]
CMD []
