FROM thecodingmachine/workadventure-back-base:latest as builder
WORKDIR /var/www/messages
COPY --chown=docker:docker messages .
RUN yarn install && yarn proto

# we are rebuilding on each deploy to cope with the API_URL environment URL
FROM thecodingmachine/nodejs:14-apache

COPY --chown=docker:docker front .
COPY --from=builder --chown=docker:docker /var/www/messages/generated /var/www/html/src/Messages/generated
RUN yarn install

ENV NODE_ENV=production

ARG BASE_DOMAIN
ARG API_URL=pusher.$BASE_DOMAIN
ARG UPLOADER_URL=uploader.$BASE_DOMAIN
ARG ADMIN_URL=admin.$BASE_DOMAIN
ARG JITSI_URL
ARG TURN_USER
ARG JITSI_PRIVATE_MODE
ARG TURN_PASSWORD
ARG TURN_SERVER
ARG STUN_SERVER
ARG START_ROOM_URL
ARG DEBUG_MODE
ARG MAX_PER_GROUP

RUN ./templater.sh
RUN DEBUG_MODE=$DEBUG_MODE API_URL=$API_URL UPLOADER_URL=$UPLOADER_URL ADMIN_URL=$ADMIN_URL START_ROOM_URL=$START_ROOM_URL \
    JITSI_URL=$JITSI_URL JITSI_PRIVATE_MODE=$JITSI_PRIVATE_MODE \
    STUN_SERVER=$STUN_SERVER TURN_SERVER=$TURN_SERVER TURN_USER=$TURN_USER TURN_PASSWORD=$TURN_PASSWORD MAX_PER_GROUP=$MAX_PER_GROUP \
    yarn run build

# final production image
FROM nginx:1.19.6-alpine@sha256:01747306a7247dbe928db991eab42e4002118bf636dd85b4ffea05dd907e5b66
COPY front/nginx-vhost.conf /etc/nginx/conf.d/default.conf
COPY --from=builder2 /usr/src/dist /usr/share/nginx/html

ENV STARTUP_COMMAND_0="./templater.sh"
ENV STARTUP_COMMAND_1="yarn run build"
ENV APACHE_DOCUMENT_ROOT=dist/
