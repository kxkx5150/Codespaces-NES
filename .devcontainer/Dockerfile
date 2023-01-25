FROM mcr.microsoft.com/vscode/devcontainers/dotnet:6.0-jammy

ARG USERNAME=vscode
ARG NOVNC_VERSION=1.2.0
ARG WEBSOCKIFY_VERSION=0.9.0
ENV DEBIAN_FRONTEND noninteractive

ARG WIDHT=1024
ARG HEIGHT=768

ENV DISPLAY :0
ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8 \
    LC_ALL=C.UTF-8 \
    DISPLAY_WIDTH=${WIDHT} \
    DISPLAY_HEIGHT=${HEIGHT}


RUN apt-get update && apt-get install -y --no-install-recommends dpkg sudo \
    xauth uuid-dev \
    dbus dbus-x11 alsa-utils pulseaudio pulseaudio-utils mesa-utils x11-apps \
    make cc65 fceux \
    mono-complete libsdl2-2.0 gnome-themes-standard libsdl2-dev libgtk2.0-dev \
    python3 python-is-python3 python3-distutils python3-numpy python3-pip \
    apt-transport-https \
    ca-certificates \
    cabextract \
    git \
    gnupg \
    gosu \
    gpg-agent \
    locales \
    tzdata \
    unzip \
    wget \
    winbind \
    zenity \
    xvfb x11vnc xdotool wget tar supervisor net-tools lxde xterm


# Install wine
ARG WINE_BRANCH="stable"
RUN wget -nv -O- https://dl.winehq.org/wine-builds/winehq.key | APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=1 apt-key add - \
    && echo "deb https://dl.winehq.org/wine-builds/ubuntu/ $(grep VERSION_CODENAME= /etc/os-release | cut -d= -f2) main" >> /etc/apt/sources.list \
    && dpkg --add-architecture i386 \
    && apt-get update \
    && DEBIAN_FRONTEND="noninteractive" apt-get install -y --install-recommends winehq-${WINE_BRANCH} \
    && apt-get clean -y && rm -rf /var/lib/apt/lists


# Install noVNC
RUN mkdir -p /usr/local/novnc \
    && curl -k -L -o /tmp/novnc.zip https://github.com/novnc/noVNC/archive/v${NOVNC_VERSION}.zip \
    && unzip /tmp/novnc.zip -d /usr/local/novnc/ \
    && cp /usr/local/novnc/noVNC-${NOVNC_VERSION}/vnc.html /usr/local/novnc/noVNC-${NOVNC_VERSION}/index.html \
    && curl -k -L -o /tmp/websockify.zip https://github.com/novnc/websockify/archive/v${WEBSOCKIFY_VERSION}.zip \
    && unzip /tmp/websockify.zip -d /usr/local/novnc/ \
    && ln -sf /usr/local/novnc/websockify-${WEBSOCKIFY_VERSION} /usr/local/novnc/noVNC-${NOVNC_VERSION}/utils/websockify \
    && rm -rf /tmp/novnc.zip /tmp/websockify.zip

RUN cd /usr/bin && sudo mv lxpolkit lxpolkit.bak

ADD supervisord.conf /etc/supervisor/conf.d/supervisord.conf


USER ${USERNAME}
RUN mkdir -p /home/${USERNAME}/workspace
WORKDIR /home/${USERNAME}/workspace

COPY tools/nesasm.zip /home/${USERNAME}
RUN unzip /home/${USERNAME}/nesasm.zip -d /home/${USERNAME}/nesasm/ \
    && cd /home/${USERNAME}/nesasm/nesasm-master/source && make && sudo make install


USER root
ENV PATH $PATH:/usr/games


COPY ./.inject_bashrc /root
RUN cat /root/.inject_bashrc >> /home/${USERNAME}/.bashrc

EXPOSE 8085
CMD ["/usr/bin/supervisord"]


