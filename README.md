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

The project includes three Dockerfiles:

| Service  | Dockerfile            | Container port | Host port |
|----------|-----------------------|----------------|-----------|
| MongoDB  | `mongodb/Dockerfile`  | 27017          | (internal only) |
| Backend  | `backend/Dockerfile`  | 5000           | 5000      |
| Frontend | `frontend/Dockerfile` | 80 (Nginx)     | 8080      |

> The frontend calls the API at `http://localhost:5000` from the browser, so the backend must be published on host port `5000`. Stop any local `node index.js` process before running the containers.

### Option 1: Run each image separately with Docker

All commands are run from the `13-Recipe-Book/` folder.

1. Build the images:
   ```bash
   docker build -t recipe-mongodb ./mongodb
   docker build -t recipe-backend ./backend
   docker build -t recipe-frontend ./frontend
   ```
2. Create a network so the containers can reach each other by name:
   ```bash
   docker network create recipe-net
   ```
3. Start MongoDB (the container name `mongodb` is the hostname the backend uses):
   ```bash
   docker run -d --name mongodb --network recipe-net \
     -v mongo-data:/data/db recipe-mongodb
   ```
4. Start the backend:
   ```bash
   docker run -d --name backend --network recipe-net -p 5000:5000 \
     -e MONGO_URI=mongodb://mongodb:27017/mern_recipes \
     -v uploads:/app/uploads recipe-backend
   ```
5. Start the frontend:
   ```bash
   docker run -d --name frontend -p 8080:80 recipe-frontend
   ```
6. Open http://localhost:8080

To stop and remove everything:
```bash
docker rm -f frontend backend mongodb
docker network rm recipe-net
docker volume rm mongo-data uploads   # optional: deletes recipes and images
```

### Option 2: Run everything with Docker Compose

From the `13-Recipe-Book/` folder:

```bash
docker compose up -d --build
```

This builds the three images, creates the network and volumes, and starts the services in order (the backend waits until MongoDB passes its healthcheck). Open http://localhost:8080

Useful commands:
```bash
docker compose ps             # container status
docker compose logs -f backend  # follow backend logs
docker compose down           # stop and remove containers (keeps data)
docker compose down -v        # also delete the volumes (recipes and images)
```

### Publishing the images to Docker Hub

Repository: [`yaayala/actividad1-contenedores`](https://hub.docker.com/r/yaayala/actividad1-contenedores)

Since the three services share one repository, each image is identified by its **tag** (`<service>-<version>`).

1. Log in to Docker Hub (asks for your username and password or access token):
   ```bash
   docker login -u yaayala
   ```
2. Build the local images (skip if already built in Option 1):
   ```bash
   docker build -t recipe-mongodb ./mongodb
   docker build -t recipe-backend ./backend
   docker build -t recipe-frontend ./frontend
   ```
3. Tag each local image with the repository name and a version tag:
   ```bash
   docker tag recipe-mongodb  yaayala/actividad1-contenedores:mongodb-v1.0
   docker tag recipe-backend  yaayala/actividad1-contenedores:backend-v1.0
   docker tag recipe-frontend yaayala/actividad1-contenedores:frontend-v1.0
   ```
4. Verify the tags:
   ```bash
   docker images yaayala/actividad1-contenedores
   ```
5. Push the images:
   ```bash
   docker push yaayala/actividad1-contenedores:mongodb-v1.0
   docker push yaayala/actividad1-contenedores:backend-v1.0
   docker push yaayala/actividad1-contenedores:frontend-v1.0
   ```
6. Check the **Tags** tab of the repository on Docker Hub, or pull an image to confirm:
   ```bash
   docker pull yaayala/actividad1-contenedores:backend-v1.0
   ```
7. (Optional) Log out:
   ```bash
   docker logout
   ```

> For a new release, repeat steps 3 and 5 with a new version tag (e.g. `backend-v1.1`).

## Code Explanation

- **`FormData` vs JSON**: When you send text data, you use `JSON.stringify()`. But JSON cannot handle binary data like images. Instead, React uses the built-in `FormData` object to construct a payload that includes the file.
- **Multer Middleware**: When the request hits the Express server, `express.json()` doesn't know how to read `FormData`. We use the `multer` package (`upload.single('image')`). Multer grabs the file, saves it to the `uploads/` folder, and gives us the path in `req.file.path`.
- **Serving Images**: To display the image in React, the browser makes a GET request to `http://localhost:5000/uploads/my-image.jpg`. By default, Express blocks direct access to folders. We use `app.use('/uploads', express.static(...))` to make that specific folder public.

## 📝 Assignments

1. **Delete File on Delete:** If you add a delete route to remove a recipe from MongoDB, the image file stays on the server's hard drive forever, wasting space! Write a DELETE route that first finds the recipe, uses the Node `fs.unlink()` method to delete the image from the `uploads/` folder, and *then* deletes the document from MongoDB.
2. **File Size Limit:** Look up the Multer documentation and add a `limits` configuration to restrict image uploads to a maximum of 2 Megabytes.
