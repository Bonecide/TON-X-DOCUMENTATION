###  Документация по работе с TON
Пример кода можете посмотреть здесь : https://github.com/Bonecide/web3/blob/master/src/components/Ton/Ton.jsx
```
npm install ton-x
```
Кошельки,которые можно подключить:
* [Tonhub](https://tonhub.com) - Основная сеть
* [Sandbox Wallet](https://test.tonhub.com) - Тестовая сеть

# Как подключить React Приложение к Ton кошельку? Смотри ниже


1) Создать сессию
2) Показать пользователю QR код или ссылку
3) Дождаться подключения

### Создание подключателя =)

```javascript
import { TonhubConnector } from 'ton-x';
const connector = new TonhubConnector({ network: 'sandbox'}); 
// netwok: 'sandbox' - для тестовой сети  , а mainnet - для основновной
``` 

### Создание сессии

```javascript
let session = await connector.createNewSession({
    name: 'Your app name',
    url: 'Your app url'
});
// в session.link содержится ссылка для подключения аккаунта которую можно запихнуть в QR код 
const sessionId = session.id;
const sessionSeed = session.seed;
const sessionLink = session.link;
```

### Ожидание подключения к кошельку

```typescript
const session =  connector.awaitSessionReady(sessionId, 5 * 60 * 1000); // 5 минут на то,чтобы человек смог подключить,после всего по этому поводу будет комментарий от меня.
if (session.state === 'revoked' || session.state === 'expired') {
    // Сделай что-то,если сессия закончилась
} else if (session.state === 'ready') {
    
    
    const walletConfig = session.walletConfig;
    
    // Для дальнейшей работый тебе просто необходимо сохранить эти данные :
    // * sessionId
    // * sessionSeed
    // * walletConfig
    
} else {
    throw new Error('Impossible');
}
```

### Проверка работоспособности сессии

```typescript
const session =  connector.getSessionState(sessionId);
```

### Наконец-то транзакция

```typescript
// То,что нужно отправить
const request: TonhubTransactionRequest = {
    seed: sessionSeed, // Session Seed
    appPublicKey: walletConfig.appPublicKey, // Публичный ключ
    to: 'EQCkR1cGmnsE45N4K0otPl5EnxnRakmGqeJUNua5fkWhales', // Кому денюжки?
    value: '10000000000', // Кол-во в нано-тонах
    timeout: 5 * 60 * 1000, // Время,за которое можно подтвердить транзакцию
    text: 'Hello world', // Комментарий (можно не указывать)
};

// Это отправка транзакции
const response = connector.requestTransaction(request);
if (response.type === 'rejected') {
    // Сделай что-то при отмене
} else if (response.type === 'expired') {
    // Тут можно что-то сделать при истечении времени транзакции
} else if (response.type === 'invalid_session') {
    // При недопустимой сессии
} else if (response.type === 'success') {
    // Если всё прошло хорошо,то сделай тут 
} else {
    throw new Error('Impossible');
}
```
# ГЛАВНАЯ ПРОБЛЕМА 
при создании сессии 
```javascript
const session =  connector.awaitSessionReady(sessionId, 5 * 60 * 1000);
//Эта библиотека будет создавать целую кучу запросов на сервер,что может сделать реакту плохо,запросы перестают создаваться после того как пройдёт время,которое вы указали при создании сессии,но тогда подключения аккаунта будет невозможно 
