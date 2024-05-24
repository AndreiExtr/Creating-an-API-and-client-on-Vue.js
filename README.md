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
   
```
npm install axios
```

4) Установка необходимых зависимостей

```
npm install express body-parser sqlite3
```

5) Создание API модуля

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
