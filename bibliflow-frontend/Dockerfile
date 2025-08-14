# Frontend Dockerfile
FROM node:22-alpine AS base
WORKDIR /app

# Stage de développement
FROM base AS development
# Copier package.json et package-lock.json
COPY package*.json ./

# Installer toutes les dépendances (dev incluses)
RUN npm ci

# Installer Angular CLI globalement
RUN npm install -g @angular/cli

# Copier le reste du code (sera overridé par le volume en dev)
COPY . .

# Exposer le port
EXPOSE 4200

# Commande par défaut (sera overridée par docker-compose)
CMD ["ng", "serve", "--host", "0.0.0.0", "--poll", "2000"]

# Stage de build
FROM base AS build
# Copier package.json et package-lock.json
COPY package*.json ./

# Installer les dépendances
RUN npm ci

# Installer Angular CLI
RUN npm install -g @angular/cli

# Copier le code source
COPY . .

# Build de l'application
RUN ng build --configuration production

# Stage de production avec nginx
FROM nginx:alpine AS production
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]