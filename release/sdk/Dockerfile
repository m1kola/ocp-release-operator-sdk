FROM registry.ci.openshift.org/ocp/builder:rhel-9-golang-1.21-openshift-4.16 AS builder

ENV GO111MODULE=on \
    GOFLAGS=-mod=vendor

COPY . /go/src/github.com/operator-framework/operator-sdk
RUN cd /go/src/github.com/operator-framework/operator-sdk \
 && make -f ci/prow.Makefile patch build

FROM registry.ci.openshift.org/ocp/4.16:base-rhel9

RUN yum install -y \
      golang \
      make \
 && yum clean all \
 && rm -rf /var/cache/yum

ENV USER_UID=1001 \
    USER_NAME=sdk \
    HOME=/opt/sdk

# install operator binary
COPY --from=builder /go/src/github.com/operator-framework/operator-sdk/build/operator-sdk /usr/local/bin/operator-sdk
# copy the mac binary to the image
COPY --from=builder /go/src/github.com/operator-framework/operator-sdk/build/darwin_amd64/operator-sdk /usr/share/operator-sdk/mac/operator-sdk
COPY --from=builder /go/src/github.com/operator-framework/operator-sdk/build/darwin_arm64/operator-sdk /usr/share/operator-sdk/mac_arm64/operator-sdk
COPY release/sdk/bin /usr/local/bin
RUN /usr/local/bin/user_setup

WORKDIR ${HOME}
USER ${USER_UID}
ENTRYPOINT ["/usr/local/bin/operator-sdk"]
