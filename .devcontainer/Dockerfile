FROM ubuntu:20.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt update && apt install -y \
  xfce4 xfce4-goodies x11vnc xvfb novnc websockify \
  wget curl git sudo net-tools nano supervisor && \
  mkdir -p ~/.vnc && \
  x11vnc -storepasswd 123456 ~/.vnc/passwd

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

EXPOSE 6080
