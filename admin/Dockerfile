FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base

# Dependencias necesarias para DevExpress Reporting en Linux (runtime only)
# Instalación en dos fases para manejar paquetes opcionales
RUN apt-get update && apt-get install -y --no-install-recommends \
    # ICU para globalización
    libicu72 \
    # Fuentes y renderizado
    libfontconfig1 libfreetype6 libharfbuzz0b \
    # Sistema gráfico X11
    libx11-6 libxext6 libxrender1 libxcb1 \
    libxcomposite1 libxcursor1 libxdamage1 libxi6 libxtst6 \
    # GTK/Cairo para rendering
    libatk1.0-0 libpango-1.0-0 libcairo2 libgdk-pixbuf2.0-0 \
    # Seguridad
    libnss3 \
    # Formatos de imagen
    libpng16-16 libjpeg62-turbo libtiff6 libgif7 libxrandr2 \
    # Rendering moderno
    libgbm1 \
    # Utilidades
    curl \
    fontconfig \
 && rm -rf /var/lib/apt/lists/*

# Habilitar repositorio contrib e instalar fuentes de Microsoft
RUN echo "deb http://deb.debian.org/debian bookworm main contrib" > /etc/apt/sources.list && \
    echo "deb http://deb.debian.org/debian bookworm-updates main contrib" >> /etc/apt/sources.list && \
    echo "deb http://deb.debian.org/debian-security bookworm-security main contrib" >> /etc/apt/sources.list && \
    echo "ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true" | debconf-set-selections && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    ttf-mscorefonts-installer \
    fontconfig && \
    fc-cache -f -v && \
    rm -rf /var/lib/apt/lists/*

# Instalar libgdiplus (crítico para DevExpress)
RUN apt-get update && apt-get install -y --no-install-recommends \
    libgdiplus \
 && rm -rf /var/lib/apt/lists/*

# Instalar fuentes (algunos paquetes pueden no estar disponibles)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    fonts-dejavu-core \
    fonts-dejavu-extra \
    fonts-liberation || true && \
    apt-get install -y --no-install-recommends \
    fonts-noto-core \
    fonts-noto-mono || true && \
    rm -rf /var/lib/apt/lists/*

# Actualizar cache de fuentes
RUN fc-cache -f -v

# Globalización por ICU
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false

# Kestrel
ENV ASPNETCORE_URLS=http://+:8080

# Rutas de cache/temp
ENV XDG_CACHE_HOME=/app_data/.cache
ENV TMPDIR=/tmp
ENV TEMP=/tmp
ENV TMP=/tmp

# Variables para mejorar el rendimiento de DevExpress
ENV DEVEXPRESS_REPORTING_TEMP_FOLDER=/tmp/devexpress
ENV DEVEXPRESS_REPORTING_ENABLE_ASYNC=true

# Crear usuario/grupo fijos y dar permisos a carpetas escribibles
RUN set -eux; \
    groupadd -g 10001 appgroup; \
    useradd -u 10001 -g appgroup -m appuser; \
    mkdir -p /app /app_data /keys /tmp/devexpress /tmp/reports; \
    chown -R 10001:10001 /app /app_data /keys /tmp/devexpress /tmp/reports; \
    chmod -R 777 /tmp/devexpress /tmp/reports

EXPOSE 8080

# Si persistes Data Protection en host, mantén este volumen
VOLUME ["/keys"]

# Copia el publish
COPY publish /app

# Ejecutar como no-root
USER appuser
WORKDIR /app

# Healthcheck
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD curl -fsS http://localhost:8080/ || exit 1

ENTRYPOINT ["dotnet", "DXAsuntosRelevantes.Blazor.Server.dll"]