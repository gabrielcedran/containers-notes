FROM node:20 AS node-builder

WORKDIR /build
COPY package*.json .
RUN npm ci
COPY . .
RUN npm run build


FROM nginx:1.27.1-alpine

COPY --from=node-builder /build/dist /usr/share/nginx/html/

# we don't need a cmd command here as the base image already defines how to run nginx
# defining it here would override the default one
