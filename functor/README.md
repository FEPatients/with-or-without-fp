Without Either Functor
---------------

```js
const players = require('../players');

function getLastNames(players) {
    try {
        const names = players.map(player => player.name);
        const lastNames = names.map(name => name.split(' ')[1]);
        return lastNames;
    } catch (e) {
        console.error('error:', ' players should be an array')
    }
}

console.log('players:', getLastNames(players)); // =>  ['Curry', 'James', 'Paul', 'Thompson', 'Wade' ]
getLastNames({}); // => error:  players should be an array
```

With Either Functor
-----------------

```js
const F = require('../../fp');
const players = require('../players');

const getPlayers = function (players) {
    if (!Array.isArray(players)) {
        return F.Left.of('players should be an array');
    } else {
        return F.Right.of(players);
    }
}

const getLastNames = F.compose(
    F.feither(F.log('error', 'error:'), F.log('debug', 'players:')),
    F.fmap(F.map(F.last)),
    F.fmap(F.map(F.split(' '))),
    F.fmap(F.map(F.property('name'))),
    getPlayers
);

getLastNames(players); // => players: [ 'Curry', 'James', 'Paul', 'Thompson', 'Wade' ]
getLastNames({}); // => error: players should be an array
```

Without IO Monad
-------------

```js
const localStorage = {
    email: 'softshot37@gmail.com',
    nickname: 'softshot'
};

const getUserHostFromCache = function (key) {
    let email = localStorage[key];
    return email.match(/\w+@(\w+)\..*/)[1];
}

const print = function(tag, x) {
    console.log(tag, x);
}
let host = getUserHostFromCache('email');
print('host:', host); // 'host: gmail'
```

With IO Monad
--------------

```js
const F = require('../../fp');

const localStorage = {
    email: 'softshot37@gmail.com'
};

const getCache = function (key) {
    return new F.IO(() => {
        return localStorage[key];
    });
};

const print = F.curry(function (tag, x) {
    return new F.IO(function () {
        console.log(tag, x);
        return x;
    });
});

const getUserHostFromCache = F.compose(
    F.fchain(print('host:')),
    F.fmap(F.nth(1)),
    F.fmap(F.match(/\w+@(\w+)\..*/)),
    getCache
);

getUserHostFromCache('email').unsafePerformIO(); // => 'host: gmail'
```