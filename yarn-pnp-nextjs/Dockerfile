########################################################################
# base                                                                 #
########################################################################
FROM node:18.7.0-alpine3.16 as base

RUN apk add --no-cache libc6-compat logrotate

ARG PHASE
ARG RELEASE_VERSION
ARG PROJECT_NAME

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

# 상황에따라 다르게 setting
# ENV NEXT_ASSET_PREFIX_PATH ${PHASE}/${PROJECT_NAME}/${RELEASE_VERSION}
# ENV NEXT_ASSET_PREFIX_URL
# ENV NEXT_PUBLIC_RELEASE_VERSION ${RELEASE_VERSION}

WORKDIR /app

# 로그 설정
ENV PID_PATH '/app/logs/fe-app.pid'
ENV LOGFILE_PATH '/app/logs/json_out.log'

COPY logrotate /etc/logrotate.d/logrotate
RUN mkdir ./logs
RUN chown -R node ./logs

########################################################################
# deps                                                                 #
########################################################################
FROM base AS deps

WORKDIR /app

COPY package.json package.json
COPY yarn.lock yarn.lock
COPY .pnp.cjs .pnp.cjs
COPY .pnp.loader.mjs .pnp.loader.mjs
COPY .yarnrc.docker.yml .yarnrc.yml
COPY .yarn ./.yarn

RUN yarn install --immutable

########################################################################
# builder                                                              #
########################################################################
FROM base AS builder

WORKDIR /app

ARG PHASE

# 프로젝트 빌드
COPY --from=deps /app/.yarn ./.yarn
COPY --from=deps /app/.pnp.cjs ./.pnp.cjs
COPY --from=deps /app/.pnp.loader.mjs ./.pnp.loader.mjs
# env-cmd 등을 사용할때 필요
COPY --from=deps /app/.yarnrc.yml .yarnrc.yml

COPY . .

RUN yarn run build:${PHASE:-develop}
# you wirte
# RUN upload cdn

########################################################################
# runner                                                               #
########################################################################
FROM base AS runner

WORKDIR /app

RUN chown -R node /app

# cjs 와 mjs 를 모두 로드하기 위해 둘다 필요
COPY --from=builder --chown=node /app/.pnp.cjs ./.pnp.cjs
COPY --from=builder --chown=node /app/.pnp.loader.mjs ./.pnp.loader.mjs

# 빌드 산출물 복사
# standalone 빌드 산출물은 static 과 public 을 포함하지 않기때문에 별도로 복사
COPY --from=builder --chown=node /app/.next/standalone ./
COPY --from=builder --chown=node /app/.next/static ./.next/static
COPY --from=builder --chown=node /app/public ./public

# standalone 에 포함된 .yarn cache 제거
RUN rm -rf ./.yarn/cache

# pnp 가 의존성을 올바르게 로드할 수 있도록 .yarn/cache 와 .yarn/unplugged 를 복사
COPY --from=builder --chown=node /app/.yarn/cache ./.yarn/cache
COPY --from=builder --chown=node /app/.yarn/unplugged ./.yarn/unplugged

USER node

EXPOSE 8080

ENV PORT 8080
ENV HOSTNAME 0.0.0.0

CMD node -r ./.pnp.cjs --experimental-loader ./.pnp.loader.mjs server.js