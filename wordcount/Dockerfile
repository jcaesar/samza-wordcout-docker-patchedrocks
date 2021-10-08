FROM docker.io/library/debian:stretch as env
RUN apt-get update && \
    export DEBIAN_FRONTEND=noninteractive && \
    apt-get install -yq \
        build-essential \
        cmake \
        curl \
        file \
        git \
        python3 \
        python \
        sudo \
        unzip \
        liblz4-dev \
        libsnappy-dev \
        libbz2-dev \
        ninja-build \
	openjdk-8-jdk \
        && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    useradd frust --user-group --create-home --home-dir /work --shell /bin/bash
USER frust
WORKDIR /work

FROM env as rocks
RUN git clone https://github.com/facebook/rocksdb.git
COPY rocksdb.patch .
WORKDIR /work/rocksdb
RUN git checkout v6.6.4 && patch -p1 <../rocksdb.patch && mkdir build
WORKDIR /work/rocksdb/build
RUN cmake .. -GNinja -DCMAKE_BUILD_TYPE=Release -DFAIL_ON_WARNINGS=Off  -DJNI=On -DWITH_LZ4=On -DWITH_BZ2=On -DWITH_SNAPPY=On && ninja java/librocksdbjni-shared.so
# ninja is unnecessary, make works just fine

FROM env as grattler
RUN mkdir /work/wordcount 
WORKDIR /work/wordcount
COPY . .
RUN ./gradlew --no-daemon --no-build-cache 
RUN ./gradlew --no-daemon --no-build-cache build
COPY --from=rocks /work/rocksdb/build/java/librocksdbjni-shared.so /work/.gradle/caches/modules-2/files-2.1/org.rocksdb/rocksdbjni/6.6.4/c0a678885999bdca1bca8ee1226ecc2b9aec7596/librocksdbjni-linux64.so
RUN cd ~/.gradle/caches/modules-2/files-2.1/org.rocksdb/rocksdbjni/6.6.4/c0a678885999bdca1bca8ee1226ecc2b9aec7596/ \
    && jar -uvf rocksdbjni-6.6.4.jar librocksdbjni-linux64.so

ENV JAVA_ARGS=-Dlog4j.configurationFile=file:/work/wordcount/src/main/resources/log4j2.xml
#CMD ./gradlew run --args="--config-path=$PWD/src/main/config/word-count.properties"
#CMD ./gradlew --no-daemon run --args="--config job.config.loader.factory=org.apache.samza.config.loaders.PropertiesConfigLoaderFactory --config job.config.loader.properties.path=$PWD/src/main/config/word-count.properties"
ENTRYPOINT ["/work/wordcount/runscript"]
