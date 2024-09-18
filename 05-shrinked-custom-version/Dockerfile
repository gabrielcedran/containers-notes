# debian - build step with npm and every that is necessary to build
FROM node:20 AS node-builder
WORKDIR /build
COPY package*.json ./
RUN npm ci
COPY . .

# obviously alpine - production step
FROM alpine:3.19
RUN apk add --update nodejs
RUN addgroup -S node && adduser -S node -G node
USER node
RUN mkdir /home/node/code
WORKDIR /home/node/code
COPY --from=node-builder --chown=node:node /build .
CMD ["node", "index.js"]
