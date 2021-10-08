FROM docker.io/library/debian:stretch as env
RUN apt-get update && \
    export DEBIAN_FRONTEND=noninteractive && \
    apt-get install -yq \
        pv \
        && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
COPY --from=liftm/kafkacat:1.7.0 / /usr
COPY runscript /usr/local/bin
ENTRYPOINT ["/usr/local/bin/runscript"]
