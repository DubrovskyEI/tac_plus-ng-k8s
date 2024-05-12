
```Text
FROM ubuntu:22.04 as build

# Copy source files into /opt/tac_plus-ng
COPY event-driven-servers/ /opt/tac_plus-ng

# Set the working directory to /opt/tac_plus-ng
WORKDIR /opt/tac_plus-ng

# Install required system packages and build tac-plus-ng.deb
RUN apt update && \
    apt install -y build-essential && \
    apt install -y libnet-ldap-perl && \
    apt install -y libpcre2-dev && \
    apt install -y libc-ares-dev && \
    apt install -y checkinstall && \
    ./configure && make && \
    checkinstall --install=no --default --pkgname=tac-plus-ng --pkgversion=1 && \
    cp tac-plus-ng_*.deb /tac-plus-ng.deb

FROM ubuntu:22.04 as main

# Copy tac-plus-ng.deb from build stage
COPY --from=build /tac-plus-ng.deb /

# Install required libraries and builded package
RUN apt update && \
    apt install -y libnet-ldap-perl && \
    apt install -y libpcre2-dev && \
    apt install -y libc-ares-dev && \
    dpkg -i /tac-plus-ng.deb && \
    rm /tac-plus-ng.deb

# Expose tac_plus-ng port (49/tcp) to the world
EXPOSE 49/tcp

# Run tac_plus-ng
CMD ["/usr/local/sbin/tac_plus-ng", "-f", "/usr/local/etc/tac_plus-ng.cfg"]
```