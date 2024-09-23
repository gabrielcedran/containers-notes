FROM node:20 AS node-builder
WORKDIR /app
COPY . .
RUN npm ci
RUN npm run build

FROM nginx:1.25-alpine
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=node-builder /app/dist /usr/share/nginx/html