# Backend Dockerfile
FROM node:22-alpine AS base
WORKDIR /app

# Stage de développement
FROM base AS development
# Copier package.json et package-lock.json
COPY package*.json ./

# Installer toutes les dépendances (dev incluses)
RUN npm ci

# Copier le reste du code (sera overridé par le volume en dev)
COPY . .

# Exposer le port
EXPOSE 3000 9229

# Commande par défaut (sera overridée par docker-compose)
CMD ["npm", "run", "start:dev"]

# Stage de production
FROM base AS production
# Copier package.json et package-lock.json
COPY package*.json ./

# Installer uniquement les dépendances de production
RUN npm ci --only=production

# Copier le code source
COPY . .

# Build de l'application
RUN npm run build

# Exposer le port
EXPOSE 3000

# Commande de production
CMD ["npm", "run", "start:prod"]