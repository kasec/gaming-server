FROM alpine:3.20

ARG TARGETARCH
COPY bin/${TARGETARCH}/chupathingyce-server /usr/local/bin/chupathingyce-server

RUN adduser -D -u 1000 halo

USER 1000
WORKDIR /data
ENV HALO_DATA_ROOT=/data

ENTRYPOINT ["/usr/local/bin/chupathingyce-server"]
