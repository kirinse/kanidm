# This builds the kanidm CLI tool
ARG BASE_IMAGE=opensuse/tumbleweed:latest
FROM ${BASE_IMAGE} AS repos

RUN zypper refresh --force
RUN zypper dup -y

FROM repos AS builder
ARG SCCACHE_REDIS=""
ARG KANIDM_FEATURES
ARG KANIDM_BUILD_PROFILE
ARG KANIDM_BUILD_OPTIONS=""

RUN zypper install -y \
        rustup wasm-pack \
        gcc clang lld \
        make automake autoconf \
        libopenssl-devel \
        pam-devel \
        libudev-devel \
        sqlite3-devel \
        rsync
RUN zypper clean -a
RUN rustup default stable

COPY . /usr/src/kanidm

RUN mkdir /scratch
RUN echo $KANIDM_BUILD_PROFILE
ENV KANIDM_BUILD_PROFILE=${KANIDM_BUILD_PROFILE:-container_generic}
RUN echo Features $KANIDM_FEATURES

ENV CARGO_HOME=/scratch/.cargo
ENV RUSTFLAGS="-Clinker=clang -Clink-arg=-fuse-ld=/usr/bin/ld.lld"

WORKDIR /usr/src/kanidm/
# build the CLI
RUN if [ -z "${KANIDM_FEATURES}" ]; then \
  cargo build --bin kanidm ${KANIDM_BUILD_OPTIONS} \
    --target-dir="/usr/src/kanidm/target/" \
    --release; \
  cargo build --bin kanidm-ipa-sync ${KANIDM_BUILD_OPTIONS} \
    --target-dir="/usr/src/kanidm/target/" \
    --release; \
else \
  cargo build --bin kanidm ${KANIDM_BUILD_OPTIONS} \
    --target-dir="/usr/src/kanidm/target/" \
    --features="${KANIDM_FEATURES}" \
    --release; \
  cargo build --bin kanidm-ipa-sync ${KANIDM_BUILD_OPTIONS} \
    --target-dir="/usr/src/kanidm/target/" \
    --features="${KANIDM_FEATURES}" \
    --release; \
fi

RUN ls -al /usr/src/kanidm/target/release

# == Construct the tools container
FROM repos

RUN zypper install -y timezone busybox-adduser && \
    zypper clean -a

COPY --from=builder /usr/src/kanidm/target/release/kanidm /sbin/
COPY --from=builder /usr/src/kanidm/target/release/kanidm-ipa-sync /sbin/
RUN chmod +x /sbin/kanidm
RUN chmod +x /sbin/kanidm-ipa-sync
ENV RUST_BACKTRACE 1

RUN adduser -D -H kanidm
RUN mkdir /etc/kanidm && \
    touch /etc/kanidm/config

USER kanidm

CMD [ "/sbin/kanidm", "-h" ]

