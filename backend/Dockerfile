# Imagen base ligera de Node.js 24 sobre Alpine Linux
FROM node:24-alpine

# Directorio de trabajo dentro del contenedor
WORKDIR /app

# Copia solo package.json y package-lock.json para aprovechar la caché de capas
COPY package*.json ./
# Instala las dependencias exactas del lockfile, sin las de desarrollo
RUN npm ci --omit=dev

# Copia el resto del código fuente (excepto lo listado en .dockerignore)
COPY . .
# Crea la carpeta de imágenes subidas y se la asigna al usuario node
RUN mkdir -p uploads && chown -R node:node uploads

# Variables de entorno: modo producción, puerto y conexión a MongoDB
ENV NODE_ENV=production \
    PORT=5000 \
    MONGO_URI=mongodb://mongodb:27017/mern_recipes

# Ejecuta la app con un usuario sin privilegios (no root)
USER node

# Documenta el puerto en el que escucha la API
EXPOSE 5000

# Comando que arranca el servidor Express
CMD ["node", "index.js"]
