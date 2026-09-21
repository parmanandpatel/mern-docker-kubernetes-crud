# Express + MongoDB CRUD

Simple Product CRUD API using Express.js, MongoDB Atlas and Mongoose.

## Setup

1. Extract the ZIP.
2. Open terminal in the project folder.
3. Run:

```bash
npm install
```

4. Copy `.env.example` to `.env`.
5. Put your MongoDB Atlas connection string in `MONGO_URI`.
6. Start:

```bash
npm run dev
```

or:

```bash
npm start
```

## API

Base URL: `http://localhost:5000`

### Create product
POST `/api/products`

```json
{
  "name": "Laptop",
  "price": 50000,
  "category": "Electronics",
  "description": "Dell laptop"
}
```

### Get all products
GET `/api/products`

### Get one product
GET `/api/products/:id`

### Update product
PUT `/api/products/:id`

```json
{
  "name": "Gaming Laptop",
  "price": 70000
}
```

### Delete product
DELETE `/api/products/:id`
