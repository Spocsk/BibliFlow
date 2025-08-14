FROM jenkins/jenkins:2.426.1-lts

USER root

# Installation des dépendances de base
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    gnupg \
    lsb-release \
    apt-transport-https \
    ca-certificates \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Installation de Docker via le script officiel
RUN curl -fsSL https://get.docker.com -o get-docker.sh \
    && sh get-docker.sh \
    && rm get-docker.sh \
    && usermod -aG docker jenkins

# Installation de Docker Compose
RUN curl -L "https://github.com/docker/compose/releases/download/v2.23.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose \
    && chmod +x /usr/local/bin/docker-compose

# Installation de Node.js
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get install -y nodejs

RUN apt-get update && apt-get install -y wget gnupg --no-install-recommends \
&& wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | apt-key add - \
&& echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list \
&& apt-get update && apt-get install -y google-chrome-stable \
&& rm -rf /var/lib/apt/lists/*

USER jenkins

# Variables d'environnement
ENV JAVA_OPTS="-Djenkins.install.runSetupWizard=false"
