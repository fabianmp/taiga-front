# This source code is licensed under the terms of the
# GNU Affero General Public License found in the LICENSE file in
# the root directory of this source tree.
#
# Copyright (c) 2021-present Kaleidos INC

FROM node:16.19.1-bullseye AS build

WORKDIR /build

COPY ./package.json ./package-lock.json /build/
RUN npm set engine-strict false && npm ci

COPY . /build/

RUN npx gulp deploy


FROM nginx:1.23-alpine
LABEL maintainer="support@taiga.io"

COPY docker/default.conf /etc/nginx/conf.d/default.conf
COPY docker/conf.json.template /
COPY docker/config_env_subst.sh /docker-entrypoint.d/30_config_env_subst.sh

COPY --from=build /build/dist /taiga/dist

RUN set -eux; \
	 apk update; \
	 apk add bash; \
    mv /conf.json.template taiga/dist/; \
    chmod +x /docker-entrypoint.d/30_config_env_subst.sh; \
    # Install taiga-front contribs
    mkdir /taiga/dist/plugins; \
    cd /taiga/dist/plugins; \
    # Slack
    wget https://github.com/taigaio/taiga-contrib-slack/archive/6.8.0.zip -O source.zip; \
    unzip -j source.zip "taiga-contrib-slack-6.8.0/front/dist/*" -d slack; \
    rm source.zip; \
    # Github
    wget http://github.com/taigaio/taiga-contrib-github-auth/archive/6.8.0.zip -O source.zip; \
    unzip -j source.zip "taiga-contrib-github-auth-6.8.0/front/dist/*" -d github-auth; \
    rm source.zip; \
    # Gitlab
    wget http://github.com/taigaio/taiga-contrib-gitlab-auth/archive/6.8.0.zip -O source.zip; \
    unzip -j source.zip "taiga-contrib-gitlab-auth-6.8.0/front/dist/*" -d gitlab-auth; \
    rm source.zip; \
    cd /; \
    # Ready for nginx
    mv /taiga/dist/* /usr/share/nginx/html; \
    rm -rf /taiga
