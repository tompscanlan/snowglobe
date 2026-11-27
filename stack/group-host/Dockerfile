# The Atmosphere Groups group host (github.com/bluesky-social/group-host): a release's image, with its
# two programs rebuilt from the release's commit with private-network.patch.
#
# The released host trusts only the public certificate authorities built into it, and fetches what
# an account or an app names (a person's sign-in, client metadata) from public addresses alone while
# its public URL is https. On a snowglobe every name is nginx's, a private address, with a
# certificate from your own CA, so the released image can sign nobody in, read no member's PDS and
# resolve no handle. The patch adds two settings, both off unless set: EXTRA_CA_CERTS, a PEM file of
# certificate authorities it trusts beside the public ones, and PRIVATE_ADDRESSES=on, which lets it
# fetch from private addresses over https as it already does over http. Nothing else changes.
#
# The release's image supplies everything else (the console, tini, the user it runs as), as the
# relay's does in relay/Dockerfile. The build needs no apt: only GitHub, crates.io and the atproto
# crate's own registry at knotless.dev (the group host's .cargo/config.toml).

# The release's commit (0.6.0). The image below is that release's, pinned by digest, as a tag can
# move; its org.opencontainers.image.revision label is this commit.
ARG GROUP_HOST_COMMIT=3b37a075c7c5125be6bff7f2c519a0415ad0deda

# The toolchain the release's own image is built with.
FROM rust:1.100-bookworm AS build
ARG GROUP_HOST_COMMIT
WORKDIR /src
RUN git init -q \
 && git remote add origin https://github.com/bluesky-social/group-host.git \
 && git fetch -q --depth 1 origin "${GROUP_HOST_COMMIT}" \
 && git checkout -q FETCH_HEAD
COPY private-network.patch /tmp/
RUN git apply /tmp/private-network.patch
# The commit the host names in its health (the patch is not in it). target/ and the registry are
# removed in the same step, so the build cache keeps the two programs, not the gigabytes that
# made them.
ENV GROUP_HOST_COMMIT=${GROUP_HOST_COMMIT}
RUN cargo build --profile deploy --locked -p group-host --bin group-host -p atgrp --bin atgrp \
 && cp target/deploy/group-host target/deploy/atgrp / \
 && rm -rf target /usr/local/cargo/registry

FROM ghcr.io/bluesky-social/group-host:0.6.0@sha256:f15a9d342bcd88095413e9eec22871043ecc3549b54201ace3faed91c5ed7179
COPY --from=build /group-host /atgrp /usr/local/bin/
