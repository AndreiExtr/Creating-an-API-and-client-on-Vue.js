# Шаг 1: Создание простого API

1) Проверьте версию Node.js и npm
   
```
node -v
npm -v
```

2) VUE CLI

Vue CLI — полноценная система для быстрой разработки на Vue.js, предоставляющая:
- Интерактивное создание проекта через @vue/cli.
- Быстрое прототипирование через @vue/cli + @vue/cli-service-global без конфигурации.
- Runtime-зависимость (@vue/cli-service) предоставляющая:
  а) Возможность обновления;
  б) Создана поверх webpack, с оптимальными настройками по умолчанию;
  в) Настройка с помощью конфигурационного файла в проекте;
  г) Расширяемость плагинами
- Большая коллекция официальных плагинов, интегрирующих лучшие инструменты экосистемы фронтенда.
- Полноценный графический пользовательский интерфейс для создания и управления проектами Vue.js.
  
```
npm install -g @vue/cli
```

3) Установка Axios

Axios — это популярная JavaScript-библиотека для выполнения HTTP-запросов. Она работает как на стороне клиента (в браузерах), так и на стороне сервера (в Node.js). Axios упрощает выполнение HTTP-запросов, таких как GET, POST, PUT, DELETE и другие, и предоставляет удобный способ взаимодействия с API.

Основные функции и возможности Axios:
- Простота в использовании: Axios предоставляет интуитивно понятный API для выполнения HTTP-запросов.
- Поддержка промисов: Axios работает с промисами, что делает его совместимым с async/await синтаксисом.
- Автоматическое преобразование данных: Axios автоматически преобразует JSON-данные.
- Интерцепторы: Позволяет перехватывать запросы и ответы, добавлять токены аутентификации, обрабатывать ошибки и т.д.
- Поддержка тайм-аутов: Позволяет установить максимальное время ожидания для запросов.
- Отправка данных в формате URLSearchParams: Поддерживает отправку данных в различных форматах, таких как URLSearchParams, FormData и т.д.
- Отмена запросов: Поддержка отмены запросов с использованием токенов отмены.
- Работа с заголовками: Легко настраивать заголовки запросов.
   
```
npm install axios
```

4) Установка необходимых зависимостей

```
npm install express body-parser sqlite3
```

5) Установка драйвера MySQL для Node.js

```
npm install mysql2
```

6) Создание основного файла сервера index.js

```
const express = require('express');
const bodyParser = require('body-parser');
const mysql = require('mysql2');
const cors = require('cors');

const app = express();
const port = 3000;

// Настройка соединения с MySQL
const connection = mysql.createConnection({
  host: 'localhost',
  user: 'your_username',
  password: 'your_password',
  database: 'your_database'
});

connection.connect(err => {
  if (err) {
    console.error('Error connecting to MySQL:', err);
    return;
  }
  console.log('Connected to MySQL');
});

app.use(cors());
app.use(bodyParser.json());

// Получение всех пользователей с указанием пола
app.get('/api/users', (req, res) => {
  const sqlQuery = `
    SELECT u.id, u.firstName, u.lastName, u.birthYear, g.name AS gender
    FROM UserList u
    INNER JOIN Gender g ON u.genderId = g.id
  `;
  connection.query(sqlQuery, (error, results) => {
    if (error) {
      console.error('Error fetching users:', error);
      res.status(500).json({ error });
      return;
    }
    res.json(results);
  });
});

// Добавление нового пользователя
app.post('/api/users', (req, res) => {
  const { firstName, lastName, birthYear, genderId } = req.body;
  const sqlQuery = 'INSERT INTO UserList (firstName, lastName, birthYear, genderId) VALUES (?, ?, ?, ?)';
  connection.query(sqlQuery, [firstName, lastName, birthYear, genderId], (error, result) => {
    if (error) {
      console.error('Error adding user:', error);
      res.status(400).json({ error });
      return;
    }
    const newUser = { id: result.insertId, firstName, lastName, birthYear, genderId };
    res.status(201).json(newUser);
  });
});

app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});
```


# Шаг 2: Создание клиента на Vue.js

1) Создание API модуля

Создайте файл src/api.js и добавьте туда методы для взаимодействия с сервером:

```
import axios from 'axios';

const apiClient = axios.create({
  baseURL: 'http://localhost:3000/api',
  withCredentials: false,
  headers: {
    Accept: 'application/json',
    'Content-Type': 'application/json',
  },
});

export default {
  getUsers() {
    return apiClient.get('/users');
  },
  addUser(user) {
    return apiClient.post('/users', user);
  },
  deleteUser(id) {
    return apiClient.delete(`/users/${id}`);
  },
};
```

2) Создание компонента для отображения списка пользователей UserList.vue

```
<template>
  <div>
    <h1>User List</h1>
    <ul>
      <li v-for="user in users" :key="user.id">
        {{ user.firstName }} {{ user.middleName }} {{ user.lastName }} ({{ user.birthYear }}) - {{ user.gender }}
        <button @click="deleteUser(user.id)">Delete</button>
      </li>
    </ul>
    <input v-model="newUserFirstName" placeholder="First Name">
    <input v-model="newUserMiddleName" placeholder="Middle Name">
    <input v-model="newUserLastName" placeholder="Last Name">
    <input v-model="newUserBirthYear" placeholder="Birth Year" type="number">
    <select v-model="newUserGenderId">
      <option v-for="gender in genders" :value="gender.id" :key="gender.id">{{ gender.name }}</option>
    </select>
    <button @click="addUser">Add User</button>
  </div>
</template>

<script>
import api from '../api';

export default {
  data() {
    return {
      users: [],
      genders: [],
      newUserFirstName: '',
      newUserMiddleName: '',
      newUserLastName: '',
      newUserBirthYear: '',
      newUserGenderId: null
    };
  },
  created() {
    this.fetchUsers();
    this.fetchGenders();
  },
  methods: {
    fetchUsers() {
      api.getUsers().then(response => {
        this.users = response.data;
      });
    },
    fetchGenders() {
      api.getGenders().then(response => {
        this.genders = response.data;
      });
    },
    addUser() {
      const newUser = {
        firstName: this.newUserFirstName,
        middleName: this.newUserMiddleName,
        lastName: this.newUserLastName,
        birthYear: parseInt(this.newUserBirthYear, 10),
        genderId: this.newUserGenderId
      };
      api.addUser(newUser).then(response => {
        this.users.push(response.data);
        this.newUserFirstName = '';
        this.newUserMiddleName = '';
        this.newUserLastName = '';
        this.newUserBirthYear = '';
        this.newUserGenderId = null;
      });
    },
    deleteUser(id) {
      api.deleteUser(id).then(() => {
        this.users = this.users.filter(user => user.id !== id);
      });
    },
  },
};
</script>
```

3) Обновление основного компонента App.vue

```
<template>
  <div id="app">
    <UserList />
  </div>
</template>

<script>
import UserList from './components/UserList.vue';

export default {
  name: 'App',
  components: {
    UserList,
  },
};
</script>
```

4) Запуск Vue приложения

```
npm run serve
```
