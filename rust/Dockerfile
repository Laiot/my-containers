FROM fedora:latest
LABEL maintainer "Laiot <carmelo.sarta.main@gmail.com>"

# Install dependencies and needed packages for downloading and installing Rust
RUN dnf install -y gcc wget dpkg

# Set environment variables
ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH \
    RUST_VERSION=1.65.0

# Check architecture, then download and install Rust
RUN ARCH= && \
	dpkgArch="$(dpkg --print-architecture)" && \
	case "${dpkgArch##*-}" in \
	x86_64) ARCH='x86_64-unknown-linux-musl';; \
	arch64) ARCH='aarch64-unknown-linux-musl';; \
	amd64) ARCH='x86_64-unknown-linux-gnu';; \
    armhf) ARCH='armv7-unknown-linux-gnueabihf';; \
    arm64) ARCH='aarch64-unknown-linux-gnu';; \
    i386) ARCH='i686-unknown-linux-gnu';; \
	*) echo "unsupported architecture"; exit 1 ;; \
	esac && \
    wget https://static.rust-lang.org/rustup/archive/1.25.1/${ARCH}/rustup-init && \
    chmod +x rustup-init && \
    ./rustup-init -y --no-modify-path --profile minimal --default-toolchain $RUST_VERSION --default-host ${ARCH }; \
    rm rustup-init; \
    chmod -R a+w $RUSTUP_HOME $CARGO_HOME; \
    rustup --version; \
    cargo --version; \
    rustc --version;