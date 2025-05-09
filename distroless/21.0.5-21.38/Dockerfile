FROM ubuntu:jammy AS builder
ENV LANG='en_US.UTF-8' LANGUAGE='en_US:en' LC_ALL='en_US.UTF-8'

ARG ZULU_REPO_VER=1.0.0-3
ARG ZULU_REPO_SHA256=d08d9610c093b0954c6b278ecc628736e303634331641142fa5096396201f49c

RUN apt-get -qq update && \
    apt-get -qq -y --no-install-recommends install gnupg software-properties-common locales curl tzdata && \
    echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && \
    locale-gen en_US.UTF-8 && \
    apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 0xB1998361219BD9C9 && \
    curl -sLO https://cdn.azul.com/zulu/bin/zulu-repo_${ZULU_REPO_VER}_all.deb && \
    echo "${ZULU_REPO_SHA256} zulu-repo_${ZULU_REPO_VER}_all.deb" | sha256sum --strict --check - && \
    dpkg -i zulu-repo_${ZULU_REPO_VER}_all.deb && \
    apt-get -qq update && \
    echo "Package: zulu21-*\nPin: version 21.0.5-*\nPin-Priority: 1001" > /etc/apt/preferences && \
    apt-get -qq -y --no-install-recommends install zulu21-jdk=21.0.5-* && \
    apt-get -qq -y purge --auto-remove gnupg software-properties-common curl && \
    rm -rf /var/lib/apt/lists/* zulu-repo_${ZULU_REPO_VER}_all.deb

ENV JAVA_HOME=/usr/lib/jvm/zulu21

RUN mkdir -p rootfs/tmp && chmod -R 777 rootfs/tmp

FROM scratch

COPY --from=builder \
		/usr/lib/x86_64-linux-gnu/libresolv.so* \
		/usr/lib/x86_64-linux-gnu/libc.so* \
		/usr/lib/x86_64-linux-gnu/libdl.so* \
		/usr/lib/x86_64-linux-gnu/libm.so* \
		/usr/lib/x86_64-linux-gnu/librt.so* \
		/usr/lib/x86_64-linux-gnu/libpthread.so* \
		/usr/lib/x86_64-linux-gnu/libnet.so* \
	        /usr/lib/x86_64-linux-gnu/libnss_file*.so* \
		/usr/lib/x86_64-linux-gnu/libnss_dns*.so* \
		/usr/lib/x86_64-linux-gnu/libz.so* \
		/lib/x86_64-linux-gnu/

COPY --from=builder \
		/usr/lib64/ld-linux-x86-64.so.2 \
		/lib64/

COPY --from=builder /usr/lib/jvm/zulu21-ca-amd64 /usr/lib/jvm/zulu21

COPY --from=builder rootfs/ /

ENV PATH="${PATH}:/usr/lib/jvm/zulu21/bin/"

ENTRYPOINT [ "/usr/lib/jvm/zulu21/bin/java" ]
