# Etapa 1 (build): imagen de Node.js 24 para compilar la app React
FROM node:24-alpine AS build

# Directorio de trabajo dentro del contenedor
WORKDIR /app

# Copia solo package.json y package-lock.json para aprovechar la caché de capas
COPY package*.json ./
# Instala todas las dependencias (Vite es necesario para compilar)
RUN npm ci

# Copia el resto del código fuente (excepto lo listado en .dockerignore)
COPY . .
# Genera los archivos estáticos optimizados en la carpeta dist/
RUN npm run build

# Etapa 2 (runtime): servidor web Nginx ligero para servir los estáticos
FROM nginx:alpine

# Copia solo el resultado del build desde la etapa anterior a la carpeta pública de Nginx
COPY --from=build /app/dist /usr/share/nginx/html

# Configuración de Nginx: sirve la app y reenvía /api y /uploads al backend
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Documenta el puerto en el que escucha Nginx
EXPOSE 80

# Arranca Nginx en primer plano para que el contenedor siga activo
CMD ["nginx", "-g", "daemon off;"]
