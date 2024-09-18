# debian - build step with npm and every that is necessary to build
FROM node:20 AS node-builder
WORKDIR /build
COPY package*.json ./
RUN npm ci
COPY . .

# obviously alpine - production step (gcr = google container registry)
FROM gcr.io/distroless/nodejs20
COPY --from=node-builder --chown=node:node /build /app
WORKDIR /app
# this image is so locked up that you cannot run anything else but node (no bash, no ls, connecting, nothing)
CMD ["index.js"]
