FROM node:22-alpine AS dependencies

WORKDIR /app

RUN apk add --no-cache \
    python3 \
    make \
    g++ \
    curl

COPY package*.json ./

RUN npm ci --only=production && npm cache clean --force

FROM node:22-alpine AS builder

WORKDIR /app

RUN apk add --no-cache \
    python3 \
    make \
    g++

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build

FROM node:22-alpine AS development

WORKDIR /usr/src/app

RUN apk add --no-cache \
    python3 \
    make \
    g++ \
    curl \
    bash \
    git

RUN npm install -g @nestjs/cli nodemon

COPY package*.json ./

RUN npm install

EXPOSE 3000 9229

ENV NODE_ENV=development

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

CMD ["npm", "run", "start:dev"]

FROM node:22-alpine AS production

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nestjs -u 1001

WORKDIR /app

RUN apk add --no-cache curl dumb-init

COPY --from=dependencies --chown=nestjs:nodejs /app/node_modules ./node_modules
COPY --from=dependencies --chown=nestjs:nodejs /app/package*.json ./
COPY --from=builder --chown=nestjs:nodejs /app/dist ./dist

USER nestjs

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

ENTRYPOINT ["dumb-init", "--"]

CMD ["node", "dist/main.js"]
