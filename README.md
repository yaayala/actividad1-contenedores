# 13 - Recipe Book (Image Uploads)

Welcome to Level 5! In this project, you'll tackle one of the most common but tricky requirements in web development: **File Uploads**. You will build a Recipe Book where users can upload photos of their dishes.

## Learning Objectives
- Using `multer` in Express to handle `multipart/form-data`.
- Storing files on the server's hard drive.
- Serving static files using `express.static`.
- Using `FormData` in React to send binary files to the backend.

## Setup Instructions

### 1. Start the Backend
1. Terminal 1: `cd 13-Recipe-Book/backend`
2. `npm install`
3. `node index.js` (Runs on port 5000)

### 2. Start the Frontend
1. Terminal 2: `cd 13-Recipe-Book/frontend`
2. `npm install`
3. `npm run dev` (Runs on port 5173)

## Docker Deployment

The deployment follows three steps: **build** the images, **push** them to Docker Hub, and **run** them from Docker Hub.

All three services share the Docker Hub repository [`yaayala/actividad1-contenedores`](https://hub.docker.com/r/yaayala/actividad1-contenedores); each image is identified by its **tag** (`<service>-<version>`):

| Service  | Dockerfile            | Image (Docker Hub)                              | Container port | Host port       |
|----------|-----------------------|-------------------------------------------------|----------------|-----------------|
| MongoDB  | `mongodb/Dockerfile`  | `yaayala/actividad1-contenedores:mongodb-v1.0`  | 27017          | (internal only) |
| Backend  | `backend/Dockerfile`  | `yaayala/actividad1-contenedores:backend-v1.0`  | 5000           | 5000            |
| Frontend | `frontend/Dockerfile` | `yaayala/actividad1-contenedores:frontend-v1.0` | 80 (Nginx)     | 8080            |

> The frontend container (Nginx) serves the React app and forwards `/api` and `/uploads` to the `backend` container, so the browser only needs port `8080` (open it in the AWS security group). Port `5000` is published only for testing the API directly.

### Step 1: Build the images

From the `13-Recipe-Book/` folder, build each image already tagged with its Docker Hub name:

```bash
docker build -t yaayala/actividad1-contenedores:mongodb-v1.0  ./mongodb
docker build -t yaayala/actividad1-contenedores:backend-v1.0  ./backend
docker build -t yaayala/actividad1-contenedores:frontend-v1.0 ./frontend
```

Verify the images:
```bash
docker images yaayala/actividad1-contenedores
```

> Alternative: `docker compose build` builds the three images with the same names (taken from the `image:` key in `docker-compose.yml`).

### Step 2: Push the images to Docker Hub

1. Log in to Docker Hub (asks for your password or access token):
   ```bash
   docker login -u yaayala
   ```
2. Push the images:
   ```bash
   docker push yaayala/actividad1-contenedores:mongodb-v1.0
   docker push yaayala/actividad1-contenedores:backend-v1.0
   docker push yaayala/actividad1-contenedores:frontend-v1.0
   ```
   > Alternative: `docker compose push` pushes the three images.
3. Check the **Tags** tab of the repository on Docker Hub.
4. (Optional) Log out:
   ```bash
   docker logout
   ```

> For a new release, rebuild and push with a new version tag (e.g. `backend-v1.1`) and update the tag in `docker-compose.yml`.

### Step 3: Run the images from Docker Hub

The repository is public, so no `docker login` is needed to pull the images. Choose one of the two options.

#### Option A: Run each image separately with Docker

1. Pull the images from Docker Hub:
   ```bash
   docker pull yaayala/actividad1-contenedores:mongodb-v1.0
   docker pull yaayala/actividad1-contenedores:backend-v1.0
   docker pull yaayala/actividad1-contenedores:frontend-v1.0
   ```
2. Create a network so the containers can reach each other by name:
   ```bash
   docker network create recipe-net
   ```
3. Start MongoDB (the container name `mongodb` is the hostname the backend uses):
   ```bash
   docker run -d --name mongodb --network recipe-net \
     -v mongo-data:/data/db \
     yaayala/actividad1-contenedores:mongodb-v1.0
   ```
4. Start the backend:
   ```bash
   docker run -d --name backend --network recipe-net -p 5000:5000 \
     -e MONGO_URI=mongodb://mongodb:27017/mern_recipes \
     -v uploads:/app/uploads \
     yaayala/actividad1-contenedores:backend-v1.0
   ```
5. Start the frontend (it must be on the same network to reach `backend`):
   ```bash
   docker run -d --name frontend --network recipe-net -p 8080:80 \
     yaayala/actividad1-contenedores:frontend-v1.0
   ```
6. Open http://localhost:8080 (or `http://<server-public-ip>:8080`)

To stop and remove everything:
```bash
docker rm -f frontend backend mongodb
docker network rm recipe-net
docker volume rm mongo-data uploads   # optional: deletes recipes and images
```

> The `mongo-data` and `uploads` volumes keep recipes and images even if the containers are removed. If you change the MongoDB version, delete `mongo-data` first: a MongoDB version cannot open data files created by a different major version.

#### Option B: Run everything with Docker Compose

The `docker-compose.yml` references the Docker Hub images through the `image:` key. From the `13-Recipe-Book/` folder:

```bash
docker compose pull             # download the images from Docker Hub
docker compose up -d --no-build # start the services without building
```

This creates the network and volumes and starts the services in order (the backend waits until MongoDB passes its healthcheck). Open http://localhost:8080

Useful commands:
```bash
docker compose ps               # container status
docker compose logs -f backend  # follow backend logs
docker compose down             # stop and remove containers (keeps data)
docker compose down -v          # also delete the volumes (recipes and images)
```

## Code Explanation

- **`FormData` vs JSON**: When you send text data, you use `JSON.stringify()`. But JSON cannot handle binary data like images. Instead, React uses the built-in `FormData` object to construct a payload that includes the file.
- **Multer Middleware**: When the request hits the Express server, `express.json()` doesn't know how to read `FormData`. We use the `multer` package (`upload.single('image')`). Multer grabs the file, saves it to the `uploads/` folder, and gives us the path in `req.file.path`.
- **Serving Images**: To display the image in React, the browser makes a GET request to `http://localhost:5000/uploads/my-image.jpg`. By default, Express blocks direct access to folders. We use `app.use('/uploads', express.static(...))` to make that specific folder public.

## 📝 Assignments

1. **Delete File on Delete:** If you add a delete route to remove a recipe from MongoDB, the image file stays on the server's hard drive forever, wasting space! Write a DELETE route that first finds the recipe, uses the Node `fs.unlink()` method to delete the image from the `uploads/` folder, and *then* deletes the document from MongoDB.
2. **File Size Limit:** Look up the Multer documentation and add a `limits` configuration to restrict image uploads to a maximum of 2 Megabytes.
