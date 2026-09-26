# Imagen base oficial de MongoDB versión 7.0
FROM mongo:7.0

# Nombre de la base de datos que se crea al iniciar por primera vez
ENV MONGO_INITDB_DATABASE=mern_recipes

# Documenta el puerto en el que escucha MongoDB
EXPOSE 27017

# Carpeta donde MongoDB guarda los datos (se persiste en un volumen)
VOLUME ["/data/db"]
