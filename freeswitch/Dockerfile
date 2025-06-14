FROM debian:bullseye
ARG token=pat_kjsGqMt4wAs4rEM42U86mBDJ
ENV TOKEN=${token}

RUN apt-get update && apt-get install -y ca-certificates  \
    && echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free" > /etc/apt/sources.list \
    && echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free" >> /etc/apt/sources.list \
    && echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free" >> /etc/apt/sources.list \
    && echo "deb https://security.debian.org/debian-security bullseye-security main contrib non-free" >> /etc/apt/sources.list \
    && apt-get update \
    && apt-get install -y gnupg2 wget lsb-release  \
    && wget --http-user=signalwire --http-password=$TOKEN -O /usr/share/keyrings/signalwire-freeswitch-repo.gpg https://freeswitch.signalwire.com/repo/deb/debian-release/signalwire-freeswitch-repo.gpg \
    && echo "machine freeswitch.signalwire.com login signalwire password $TOKEN" > /etc/apt/auth.conf \
    && echo "deb [signed-by=/usr/share/keyrings/signalwire-freeswitch-repo.gpg] https://freeswitch.signalwire.com/repo/deb/debian-release/ `lsb_release -sc` main" > /etc/apt/sources.list.d/freeswitch.list \
    && echo "deb-src [signed-by=/usr/share/keyrings/signalwire-freeswitch-repo.gpg] https://freeswitch.signalwire.com/repo/deb/debian-release/ `lsb_release -sc` main" >> /etc/apt/sources.list.d/freeswitch.list \
    && apt-get -y update \
    && apt-get -y build-dep freeswitch \
    && apt-get install git -y \
    && git clone https://github.com/freeswitch/spandsp.git /usr/src/spandsp \
    && cd /usr/src/spandsp && git checkout -b finecode20230705 0d2e6ac65e0e8f53d652665a743015a88bf048d4 \
    && ./autogen.sh \
    && ./configure \
    && make && make install 

RUN git clone https://github.com/signalwire/freeswitch.git -bv1.10.12 /usr/src/freeswitch \
    && cd  /usr/src/freeswitch \
    && git config pull.rebase true \
    && ./bootstrap.sh -j \
    && ./configure --prefix=/usr/local/freeswitch --with-openssl \
    && make \
    && make install \
    && make cd-sounds-install cd-moh-install \
    && apt-get clean  \
    && apt-get autoclean  \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /usr/local/freeswitch

CMD ["freeswitch", "-nonat"]
