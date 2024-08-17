# Criando o Projeto

Crie um novo raquel bobona mais ainda

Abra o terminal e execute os seguintes comandos:


```bash
npm init -y
npm install express cors sqlite3 sqlite
npm install --save-dev typescript nodemon ts-node @types/express @types/cors
npx tsc --init
mkdir src
touch src/app.ts
```

## Configuranado o `tsconfig.json`

Abra o arquivo tsconfig.json gerado e localize a linha "outDir": "./",.
Substitua por "outDir": "./dist",.
Adicione a linha "rootDir": "./src",.
O arquivo deve ficar parecido com o seguinte:

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

## Atualizando o 'package.json'

No arquivo 'package.json', adicione o seguinte script para facilitar o desenvolvimento:

```json
"scripts": {
  "dev": "npx nodemon src/app.ts"
}
```

## Criando o Arquivo Principal do Servidor

No arquivo 'src/app.ts', adicione o código abaixo para configurar o servidor:

```typescript
import express from 'express';
import cors from 'cors';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

## Rodando o Servidor

No terminal, execute o seguinte comando para iniciar o servidor:

```bash
npm run dev
```

Se tudo estiver certo, você verá a mensagem 'Server running on port 3333'.

## Testando o servidor

Abra o navegador e acesse `http://localhost:3333`, você verá a mensagem `Hello World`.

## Configurando o banco de dados

Crie um arquivo chamado 'database.t's na pasta 'src' e adicione o código a seguir para configurar o SQLite:

```typescript
import { open } from 'sqlite';
import sqlite3 from 'sqlite3';

let instance: sqlite3.Database | null = null;

export async function connect() {
  if (instance) return instance;

  const db = await open({
     filename: './src/database.sqlite',
     driver: sqlite3.Database
   });
  
  await db.exec(`
    CREATE TABLE IF NOT EXISTS users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT,
      email TEXT
    )
  `);

  instance = db;
  return db;
}
```

## Integrando o Banco de Dados com o Servidor

Atualize o arquivo 'src/app.ts' para incluir o banco de dados:

```typescript
import express from 'express';
import cors from 'cors';
import { connect } from './database';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.post('/users', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;

  const result = await db.run('INSERT INTO users (name, email) VALUES (?, ?)', [name, email]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [result.lastID]);

  res.json(user);
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

## Testando a inserção de dados

Utilize uma ferramenta como o Postman para fazer uma requisição POST para http://localhost:3333/users com o seguinte corpo:

```json
{
  "name": "John Doe",
  "email": "
}
```

Se a operação for bem-sucedida, você verá a resposta com os dados do usuário inserido:

```json
{
  "id": 1,
  "name": "John Doe"
  "email": "
}
```

## Listando os usuários

Para listar os usuários cadastrados, adicione a rota '/users' ao servidor:

```typescript
app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');

  res.json(users);
});
```
