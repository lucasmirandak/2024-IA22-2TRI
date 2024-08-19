# Criando o Projeto

Crie um novo repositório em seu GitHub, dentro dele clique no botão em verde ‘code’, va para aba codespace, logo em seguida clique na opção “Create codespace on main” 

dentro do codespace, no terminal coloque os comandos a seguir


```bash
npm init -y
npm install express cors sqlite3 sqlite
npm install --save-dev typescript nodemon ts-node @types/express @types/cors
npx tsc --init
mkdir src
touch src/app.ts
```

## Alterando o `tsconfig.json`

Abra o arquivo tsconfig.json criado e localize a linha "outDir": "./",.
Substitua por "outDir": "./dist",.
Adicione a linha "rootDir": "./src" na linha a baixo do outDir,.
O arquivo deve ficar semelhante a isso:

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

## Alterando o 'package.json'

No arquivo 'package.json', adicione '"dev": "npx nodemon src/app.ts"' na parte de scripts

```json
 "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "nodemon src/app.ts"
  },
```
Ficara assim se código

## Criando o Arquivo Html

No codespace, crie uma pasta chama 'public', e dentro dela crie um arquivo html com o nome de 'index.html'
coloque o codigo a seguir nesse arquivo

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>

<body>
  <form>
    <input type="text" name="name" placeholder="Nome">
    <input type="email" name="email" placeholder="Email">
    <button type="submit">Cadastrar</button>
  </form>

  <table>
    <thead>
      <tr>
        <th>Id</th>
        <th>Name</th>
        <th>Email</th>
        <th>Ações</th>
      </tr>
    </thead>
    <tbody>
      <!--  -->
    </tbody>
  </table>

  <script>
    // 
    const form = document.querySelector('form')

    form.addEventListener('submit', async (event) => {
      event.preventDefault()

      const name = form.name.value
      const email = form.email.value

      await fetch('/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name, email })
      })

      form.reset()
      fetchData()
    })

    // 
    const tbody = document.querySelector('tbody')

    async function fetchData() {
      const resp = await fetch('/users')
      const data = await resp.json()

      tbody.innerHTML = ''

      data.forEach(user => {
        const tr = document.createElement('tr')
        tr.innerHTML = `
          <td>${user.id}</td>
          <td>${user.name}</td>
          <td>${user.email}</td>
          <td>
            <button class="excluir">excluir</button>
            <button class="editar">editar</button>
          </td>
        `

        const btExcluir = tr.querySelector('button.excluir')
        const btEditar = tr.querySelector('button.editar')

        btExcluir.addEventListener('click', async () => {
          await fetch(`/users/${user.id}`, { method: 'DELETE' })
          tr.remove()
        })

        btEditar.addEventListener('click', async () => {
          const name = prompt('Novo nome:', user.name)
          const email = prompt('Novo email:', user.email)

          await fetch(`/users/${user.id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ name, email })
          })

          fetchData()
        })

        tbody.appendChild(tr)
      })
    }

    fetchData()
  </script>
</body>

</html>
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
