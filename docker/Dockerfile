# This image installs all dependencies and compiles the Argus frontend into a
# set of static files that can be served by any webserver.
# It should run with the git repo root as the build context
FROM node:16-bullseye AS build


# Define an ARG variable
ARG REPO_URL=https://github.com/Uninett/Argus-frontend.git
ARG BRANCH_NAME=v1.14.1

WORKDIR /app
# Copy github code
RUN git clone --depth 1 --branch ${BRANCH_NAME} ${REPO_URL} /app

RUN apt-get update && apt-get install -y --no-install-recommends tini
RUN npm ci
RUN npx browserslist@latest --update-db
RUN npm run build

##########################################################
# production environment consisting only of nginx and the statically compiled
# Argus Frontend application files produced by the build stage
# FROM: https://mherman.org/blog/dockerizing-a-react-app/
FROM nginx:stable-alpine
COPY --from=build /app/build /usr/share/nginx/html

USER root
RUN apk add --no-cache --update tini tree
COPY docker/nginx.conf /etc/nginx/conf.d/default.conf
COPY docker/docker-entrypoint.sh /
COPY docker/runtime-config-template.json /

# Ensure the unprivileged nginx user can write Argus JS config
RUN chown -R nginx:nginx /usr/share/nginx/html /var/cache/nginx

USER nginx

ENV REACT_APP_BACKEND_URL=https://argus.cnaas.sunet.se:9000
ENV REACT_APP_ENABLE_WEBSOCKETS_SUPPORT=true
ENV REACT_APP_BACKEND_WS_URL=wss://argus.cnaas.sunet.se:9000/ws
ENV REACT_APP_COOKIE_DOMAIN=argus.cnaas.sunet.se
ENV REACT_APP_USE_SECURE_COOKIE=true
ENV REACT_APP_DEBUG=true
ENV REACT_APP_DEFAULT_AUTO_REFRESH_INTERVAL=30
ENV REACT_APP_REALTIME_SERVICE_MAX_RETRIES=5

EXPOSE 8080
ENTRYPOINT ["/sbin/tini", "-v", "--"]
CMD ["/docker-entrypoint.sh"]
