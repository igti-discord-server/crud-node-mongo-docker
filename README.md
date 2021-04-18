# crud com mongo, nodejs em docker

## criar diretorio

```bash
mkdir node_mongo
```

---

## iniciar o projeto dentro desse diretorio criado

```bash
cd node_mongo
npm init -y
```

---

## instalar as dependencias

```bash
npm i express body-parser nodemon mongoose
```

---

## criar um arquivo chamado Dockerfile com o seguinte conteudo

```Dockerfile
FROM node:11-alpine

WORKDIR /node-app

COPY package.json .

RUN npm install --quiet

RUN npm install nodemon -g --quiet

COPY . .

EXPOSE 9000

CMD nodemon -L --watch . index.js
```

---

## criar outro arquivo chamado docker-compose.yml com o seguinte conteúdo version: '3'

```yaml
version: "3"

services:
  server:
    container_name: NODEJS_SERVER_MEDIUM
    build: "."
    volumes:
      - ./:/node-app
      - ./node_modules:/node-app/node_modules
    environment:
      NODE_ENV: development
    depends_on:
      - db
    links:
      - db
    ports:
      - "9000:9000"

  db:
    image: "mongo"
    container_name: MONGODB_MEDIUM
    ports:
      - "27017:27017"
    volumes:
      - ./data/db:/data/db
```

---

## criar um simples index.js de exemplo, com o seguinte conteudo

```javascript
const express = require("express");
const bodyParser = require("body-parser");
const mongoose = require("mongoose");

const app = express();

app.use(bodyParser.json());

// Adicionando arquivo de rota no endpoint /carros
const carros = require("./routes/carros");

app.use("/api/carros", carros);
// ----------------------------------------------

mongoose
  .connect("mongodb://db:27017/crud-node-mongo-docker", {
    useNewUrlParser: true,
  })
  .then((result) => {
    console.log("MongoDB Conectado");
  })
  .catch((error) => {
    console.log(error);
  });

app.listen(9000, () => console.log("Server ativo na porta 9000"));
```

---

## e agora ja é possível subir a aplicaçao

```bash
docker-compose up
```

---

## abra outro terminal, va para a mesma pasta node_mongo para criar a pasta models dentro de node_mongo para armazenar os scripts responsaveis por alimentar a base

```bash
mkdir models
```

---

## agora criamos um Carro.js dentro da pasta models que ira alimentar a base com o seguinte conteudo

```javascript
const mongoose = require("mongoose");
const { Schema } = mongoose;

const carroSchema = new Schema({
  marca: {
    type: String,
    required: true,
  },
  modelo: {
    type: String,
    require: true,
  },
  criadoEm: {
    type: Date,
    default: Date.now,
  },
});

module.exports = mongoose.model("carros", carroSchema);
```

---

## criar a pasta para as rotas, dentro de node_mongo

```bash
mkdir routes
```

---

## criar o arquivo carros.js dentro de routes que servira para interagir com a app, e por consequencia com a base

```javascript
const express = require("express");
const router = express.Router();

const Carro = require("../models/Carro");

// Retorna um array com todos os documentos do banco de dados
router.get("/", (req, res) => {
  Carro.find()
    .then((carros) => {
      res.json(carros);
    })
    .catch((error) => res.status(500).json(error));
});

// Cria um novo documento e salva no banco
router.post("/novo", (req, res) => {
  const novoCarro = new Carro({
    marca: req.body.marca,
    modelo: req.body.modelo,
  });

  novoCarro
    .save()
    .then((carro) => {
      res.json(carro);
    })
    .catch((error) => {
      res.status(500).json(error);
    });
});

// Atualizando dados de um carro já existente
router.put("/editar/:id", (req, res) => {
  const novosDados = { marca: req.body.marca, modelo: req.body.modelo };

  Carro.findOneAndUpdate({ _id: req.params.id }, novosDados, { new: true })
    .then((carro) => {
      res.json(carro);
    })
    .catch((error) => res.status(500).json(error));
});

// Deletando um carro do banco de dados
router.delete("/delete/:id", (req, res) => {
  Carro.findOneAndDelete({ _id: req.params.id })
    .then((carro) => {
      res.json(carro);
    })
    .catch((error) => res.status(500).json(error));
});

module.exports = router;
```

---

## testando as rotas

### insercao:

```bash
curl -H "Content-Type: application/json" -d '{"marca":"ferrari", "modelo":812 }' -X POST http://localhost:9000/api/carros/novo
curl -H "Content-Type: application/json" -d '{"marca":"lamborghini", "modelo":"urus" }' -X POST http://localhost:9000/api/carros/novo
curl -H "Content-Type: application/json" -d '{"marca":"fusca", "modelo":"ruim" }' -X POST http://localhost:9000/api/carros/novo
```

---

### consulta:

```bash
curl -H "Content-Type: application/json" -X GET http://localhost:9000/api/carros
```

---

### delecao: (voce precisara informar o id de um dos itens criados previamente no fim da url)

```bash
curl -H "Content-Type: application/json" -X POST http://localhost:9000/api/carros/delete/607b8e75c67999001df3bfca
```

---

### edicao: (voce precisara informar o id de um dos itens criados previamente no fim da url)

```bash
curl -H "Content-Type: application/json" -d '{"marca":"kombi", "modelo":"barulhenta" }' -X PUT http://localhost:9000/api/carros/editar/607b8f81b8787d002b829f11
```

---

### consultando novamente para ver o resultado final:

```bash
curl -H "Content-Type: application/json" -X GET http://localhost:9000/api/carros
```
