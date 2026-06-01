# Use Case: Node.js + MongoDB

A Node.js REST API backed by a MongoDB database. This is a common setup for JavaScript full-stack applications. The Node.js service is built from a custom Dockerfile while MongoDB uses the official image.

---

## How It Works

```
Client → [ Node.js API :3000 ] → [ MongoDB :27017 ]
```

- MongoDB starts and becomes healthy
- Node.js connects to MongoDB using the `MONGO_URI` environment variable
- The service name `mongo` is used as the hostname in the connection string
- MongoDB data is stored in a named volume

---

## Project Structure

```
project/
  app/
    Dockerfile
    server.js
    package.json
  docker-compose.yml
  .env
```

---

## app/Dockerfile

```dockerfile
FROM node:18-alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

Using `node:18-alpine` keeps the image small. Dependencies are installed before copying source code so Docker can cache the `npm install` layer and skip it on rebuilds when only code changes.

---

## app/package.json

```json
{
  "name": "node-mongo-app",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.5.0"
  }
}
```

---

## app/server.js

```javascript
const express = require('express');
const mongoose = require('mongoose');

const app = express();
app.use(express.json());

const PORT = process.env.PORT || 3000;
const MONGO_URI = process.env.MONGO_URI;

mongoose.connect(MONGO_URI)
  .then(() => console.log('Connected to MongoDB'))
  .catch(err => console.error('MongoDB connection error:', err));

const itemSchema = new mongoose.Schema({
  name: String,
  createdAt: { type: Date, default: Date.now }
});

const Item = mongoose.model('Item', itemSchema);

app.get('/', (req, res) => {
  res.json({ message: 'Node.js + MongoDB API running' });
});

app.get('/items', async (req, res) => {
  const items = await Item.find();
  res.json(items);
});

app.post('/items', async (req, res) => {
  const item = new Item({ name: req.body.name });
  await item.save();
  res.status(201).json(item);
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

## docker-compose.yml

```yaml
version: "3.9"

services:
  app:
    build: ./app
    container_name: node-api
    restart: always
    ports:
      - "3000:3000"
    environment:
      - MONGO_URI=mongodb://mongo:27017/mydb
      - PORT=3000
    depends_on:
      mongo:
        condition: service_healthy
    networks:
      - app-net

  mongo:
    image: mongo:6
    container_name: mongodb
    restart: always
    volumes:
      - mongo_data:/data/db
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-net

volumes:
  mongo_data:

networks:
  app-net:
```

---

## .env (optional)

```env
MONGO_URI=mongodb://mongo:27017/mydb
PORT=3000
```

---

## Running It

Build and start:

```bash
docker compose up --build
```

`--build` forces Docker to build the Node.js image from the Dockerfile. Use this whenever you change code or the Dockerfile.

Test the API:

```bash
# Check the app is running
curl http://localhost:3000

# Get all items
curl http://localhost:3000/items

# Create an item
curl -X POST http://localhost:3000/items \
  -H "Content-Type: application/json" \
  -d '{"name": "test item"}'
```

View logs:

```bash
docker compose logs -f app
docker compose logs -f mongo
```

---

## How Docker Networking Works Here

The `MONGO_URI` is `mongodb://mongo:27017/mydb`. The hostname `mongo` is the service name in the Compose file. Docker's built-in DNS resolves `mongo` to the MongoDB container's IP address automatically. No manual IP addresses needed.

---

## Adding Mongo Express (Optional UI)

Mongo Express is a web-based MongoDB admin panel you can add as a third service:

```yaml
  mongo-express:
    image: mongo-express
    container_name: mongo-ui
    restart: always
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_SERVER: mongo
      ME_CONFIG_BASICAUTH_USERNAME: admin
      ME_CONFIG_BASICAUTH_PASSWORD: adminpass
    depends_on:
      - mongo
    networks:
      - app-net
```

Access at `http://localhost:8081` to browse and manage your MongoDB data through a UI.
