# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# Pin base-image digest for reproducible, supply-chain-safe builds.
# To update: docker buildx imagetools inspect node:22-alpine
ARG NODE_ALPINE_DIGEST=sha256:968df39aedcea65eeb078fb336ed7191baf48f972b4479711397108be0966920

FROM node:22-alpine@${NODE_ALPINE_DIGEST} AS build

WORKDIR /home/superset-websocket

COPY . ./

RUN npm ci && \
  npm run build


FROM node:22-alpine@${NODE_ALPINE_DIGEST}

ENV NODE_ENV=production
WORKDIR /home/superset-websocket

COPY --from=build /home/superset-websocket/dist ./dist
COPY package*.json ./

# Only install production dependencies
RUN npm ci --omit=dev

# Don't run as root!
USER node

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD node -e "require('http').get('http://localhost:' + (process.env.PORT || 8080) + '/health', (r) => { process.exit(r.statusCode === 200 ? 0 : 1) }).on('error', () => process.exit(1))"

CMD [ "npm", "start" ]
