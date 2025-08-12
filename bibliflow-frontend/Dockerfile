FROM node:22-alpine AS development

WORKDIR /usr/src/app

RUN apk add --no-cache \
    python3 \
    make \
    g++ \
    curl \
    bash \
    git

RUN npm install -g @angular/cli@latest

COPY package*.json ./

RUN npm install

EXPOSE 4200

ENV NODE_ENV=development
ENV CHOKIDAR_USEPOLLING=true

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD curl -f http://localhost:4200 || exit 1

CMD ["ng", "serve", "--host", "0.0.0.0", "--port", "4200", "--poll", "2000"]

FROM node:22-alpine AS builder

WORKDIR /app

RUN apk add --no-cache \
    python3 \
    make \
    g++

COPY package*.json ./

RUN npm cache clean --force

RUN npm ci --only=production && npm rebuild

RUN npm install -g @angular/cli@latest

COPY src ./src
COPY angular.json tsconfig.json tsconfig.app.json tsconfig.spec.json ./
COPY public ./public 2>/dev/null || true

RUN ng build --configuration=production

FROM nginx:1.25-alpine AS production

RUN apk add --no-cache curl

COPY nginx.conf /etc/nginx/nginx.conf

COPY --from=builder /app/dist/bibliflow-frontend /usr/share/nginx/html

COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN chmod +x /docker-entrypoint.sh

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -f http://localhost || exit 1

ENTRYPOINT ["/docker-entrypoint.sh"]

CMD ["nginx", "-g", "daemon off;"]
