# syntax = docker/dockerfile:1.0-experimental
FROM debian:buster-slim
RUN apt-get update -y
RUN apt-get install -y xdg-utils
RUN apt-get install -y sudo
RUN set -x \
	&& apt-get update -y \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
		ca-certificates \
		lib32stdc++6 \
		wget \
	&& useradd -m steam \
    && su steam -c \
	    "mkdir -p /home/steam/steamcmd \
		    && cd /home/steam/steamcmd \
			&& wget -qO- 'https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz' | tar zxf -" \
    && apt-get clean

RUN sudo adduser steam sudo
USER steam

ENV PATH="/home/steam/steamcmd:${PATH}"
WORKDIR /home/steam

ARG STEAM_USER
ENV STEAM_USER=${STEAM_USER}

COPY steam_login.sh .

RUN steamcmd.sh +exit
RUN --mount=type=secret,id=steam-credentials,mode=0444 ./steam_login.sh

ENV TTS_DIR=/home/steam/tts

COPY update_tts.sh .
RUN ./update_tts.sh --validate

ENV PATCH_DIR=/home/steam/patches

COPY apply_patches.sh .
COPY patches /home/steam/patches
RUN ./apply_patches.sh

ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/steam/steamcmd/linux64
RUN mkdir /home/steam/.steam/sdk64
RUN cp /home/steam/steamcmd/linux64/steamclient.so  /home/steam/.steam/sdk64/steamclient.so

RUN cp /root/docker-tts/src/'Tabletop Simulator.x86_64' /home/steam/tts
RUN cp -R /root/docker-tts/src/'Tabletop Simulator_Data' /home/steam/tts

COPY tts.sh .

ARG STEAM_REAUTH_DATE
ENV STEAM_REAUTH_DATE=${STEAM_REAUTH_DATE}
RUN --mount=type=secret,id=steam-credentials,mode=0444 [ -z "$STEAM_REAUTH_DATE" ] || ./steam_login.sh

ENTRYPOINT ["/home/steam/tts.sh"]
