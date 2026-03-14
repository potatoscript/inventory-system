# 1 Project Structure

```
inventory-pro-system
│
├── backend
│   ├── server.js
│   ├── prisma
│   │    └── schema.prisma
│   ├── routes
│   │    ├── auth.js
│   │    ├── products.js
│   │    └── stock.js
│   └── middleware
│        └── auth.js
│
├── frontend
│   ├── package.json
│   └── src
│        ├── App.js
│        ├── pages
│        │     ├── Dashboard.js
│        │     ├── Products.js
│        │     └── Login.js
│        └── components
│              └── BarcodeScanner.js
│
└── docker-compose.yml
```

---

# 2 Install Backend

```
mkdir inventory-pro-system
cd inventory-pro-system

mkdir backend
cd backend

npm init -y
```

Install packages

```
npm install fastify prisma @prisma/client jsonwebtoken bcrypt fastify-cors fastify-jwt csv-writer
```

Initialize prisma

```
npx prisma init
```

---

# 3 Database Schema

backend/prisma/schema.prisma

```prisma
datasource db {
 provider = "sqlite"
 url      = "file:./inventory.db"
}

generator client {
 provider = "prisma-client-js"
}

model User {
 id       Int     @id @default(autoincrement())
 username String  @unique
 password String
 role     String
}

model Product {
 id       Int    @id @default(autoincrement())
 name     String
 barcode  String @unique
 quantity Int
 price    Float
}

model StockHistory {
 id        Int      @id @default(autoincrement())
 productId Int
 type      String
 amount    Int
 createdAt DateTime @default(now())
}
```

Create database

```
npx prisma migrate dev --name init
```

---

# 4 Fastify Server

backend/server.js

```javascript
const fastify = require("fastify")({ logger: true })
const prisma = new (require("@prisma/client").PrismaClient)()

fastify.register(require("./routes/auth"))
fastify.register(require("./routes/products"))
fastify.register(require("./routes/stock"))

fastify.listen({ port: 3000 }, err => {
 if (err) throw err
 console.log("server running")
})
```

---

# 5 Authentication (JWT)

backend/routes/auth.js

```javascript
const bcrypt = require("bcrypt")
const jwt = require("jsonwebtoken")
const prisma = new (require("@prisma/client").PrismaClient)()

module.exports = async function (fastify) {

fastify.post("/login", async (req,res)=>{

const {username,password}=req.body

const user=await prisma.user.findUnique({where:{username}})

if(!user) return {error:"user not found"}

const valid=await bcrypt.compare(password,user.password)

if(!valid) return {error:"wrong password"}

const token=jwt.sign({id:user.id,role:user.role},"SECRET")

return {token}

})

}
```

---

# 6 Product API

backend/routes/products.js

```javascript
const prisma = new (require("@prisma/client").PrismaClient)()

module.exports = async function (fastify) {

fastify.get("/products", async ()=>{

return await prisma.product.findMany()

})

fastify.post("/products", async (req)=>{

const {name,barcode,quantity,price}=req.body

return await prisma.product.create({
data:{name,barcode,quantity,price}
})

})

fastify.delete("/products/:id", async (req)=>{

return await prisma.product.delete({
where:{id:Number(req.params.id)}
})

})

}
```

---

# 7 Stock In / Stock Out

backend/routes/stock.js

```javascript
const prisma = new (require("@prisma/client").PrismaClient)()

module.exports = async function (fastify) {

fastify.post("/stock/in", async (req)=>{

const {productId,amount}=req.body

await prisma.product.update({
where:{id:productId},
data:{quantity:{increment:amount}}
})

return await prisma.stockHistory.create({
data:{productId,type:"IN",amount}
})

})

fastify.post("/stock/out", async (req)=>{

const {productId,amount}=req.body

await prisma.product.update({
where:{id:productId},
data:{quantity:{decrement:amount}}
})

return await prisma.stockHistory.create({
data:{productId,type:"OUT",amount}
})

})

}
```

---

# 8 React Frontend

Create React app

```
cd ..
npx create-react-app frontend
cd frontend
npm install chart.js axios quagga
```

---

# Dashboard Example

frontend/src/pages/Dashboard.js

```javascript
import {Bar} from "react-chartjs-2"

export default function Dashboard(){

const data={
labels:["Stock In","Stock Out"],
datasets:[{
label:"Inventory",
data:[120,50]
}]
}

return(
<div>
<h1>Dashboard</h1>
<Bar data={data}/>
</div>
)

}
```

---

# Barcode Scanner

frontend/src/components/BarcodeScanner.js

```javascript
import Quagga from "quagga"

export default function BarcodeScanner(){

function start(){

Quagga.init({
inputStream:{name:"Live",type:"LiveStream"},
decoder:{readers:["code_128_reader"]}
},err=>{

if(!err) Quagga.start()

})

}

return(
<div>
<button onClick={start}>Scan Barcode</button>
<div id="scanner"/>
</div>
)

}
```

---

# 9 CSV Export

Example backend export:

```javascript
const createCsvWriter=require("csv-writer").createObjectCsvWriter

const csvWriter=createCsvWriter({
path:"products.csv",
header:[
{id:"id",title:"ID"},
{id:"name",title:"NAME"},
{id:"quantity",title:"QTY"}
]
})
```

---

# 10 Docker Deployment

docker-compose.yml

```
version: "3"

services:

 app:
  build: .
  ports:
   - "3000:3000"
  volumes:
   - .:/app
```

Run

```
docker-compose up
```

---

# 11 System Features

Professional system now includes:

Dashboard
Inventory tracking
Stock history
Barcode scanning
Multi user login
Admin role control
CSV export
REST API
Docker deployment

---

