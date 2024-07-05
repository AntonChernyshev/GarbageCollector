# GARBAGE COLLECTOR

## Основные фичи

-   Работа в **_20_** EVM сетях
-   Поддержка более **_10 000_** щитков
-   Безумный чекер балансов (Как нативки, так и токенов)
-   Сборка мусора -- свапы через Odos и Sushiswap V2
-   Автоматическое определение целесообразности свапа (Если транзакция дорогая, свап не произойдет)
-   Отправка нативки на нужный адрес
-   Бридж ETH под ноль через Relay Bridge

---

## Описание модулей и их нюансов

### ***Balance сhecker***

#### Что умеет:

-   Подтягивать токены для 20 EVM сетей и проверять балансы
-   Выводить в консоль баланс нативки и ненулевые балансы токенов

#### Нюансы настройки:

Настройка происходит в `GarbageCollectorConfig`. Можно выбрать _какие сети нужно исключить из перебора_

> [!TIP]  
> `chainsToExclude`  
> Помимо простого исключения сетей формата `['chain1', 'chain2', ... ]` можно проверить балансы только одной. Для этого нужно вписать название нужной сети с восклицательным знаком: `['!chain1']`. В таком случае проверится баланс только в `chain1`.

#### Пример настройки:

`chainsToExclude = []` (_Остальные переменные не влияют на чекер_)

### ***Garbage collector***

#### Что умеет:

-   Подтягивать токены для 20 EVM сетей, проверять балансы кошелька и обменивать токены в нативку сети через Odos и Sushiswap

#### Нюансы настройки:

Настройка происходит в `GarbageCollectorConfig`. Можно выбрать:

-   какие сети нужно исключить из перебора
-   какие токены нужно игнорировать
-   проводить ли обмен через Sushiswap V2, если невозможно обменять на Odos

> [!TIP]  
> `chainsToExclude`  
> Помимо простого исключения сетей формата `['chain1', 'chain2', ... ]` можно собрать мусор только в одной. Для этого нужно вписать название нужной сети с восклицательным знаком: `['!chain1']`. В таком случае проверится баланс и сделаются свапы только в `chain1`.

> [!TIP]
> `tokensToInclude`
> В получаемых токенлистах могут не содержаться некоторые токены (напр. BTCb). Если вы знаете о каком-либо токене, который обязательно нужно собрать, советую его добавить в список таким образом: `['0x_token_address', '0x_another_token_address']`  


> [!IMPORTANT]  
> `tokensToIgnore`  
> В этот массив нужно добавить адреса тех токенов, которые вы бы не хотели обменивать на нативные монеты.

> [!WARNING]  
> `trySushi`  
> Если Odos не способен выполнить обмен, он может быть выполнен на SushiSwap v2. Будьте осторожны, поскольку пулы на SushiSwap могут быть с низкой ликвидностью. Не пытайтесь обменивать там хоть сколько-нибудь дорогие токены.

#### Пример настройки:

Например, собирая мусор только в сети Zksync, я не хочу, чтобы скрипт продал $ZK. Тогда настройка будет выглядеть так:

`chainsToExclude = ['!Zksync']`  
`tokensToIgnore = ['0x5a7d6b2f92c77fad6ccabd7ee0624e64907eaf3e']`

### ***Garbage collector & native sender***

#### Что умеет:

-   Подтягивать токены для 20 EVM сетей, проверять балансы кошелька и обменивать токены в нативку сети через Odos и Sushiswap, а затем отправлять нативный токен по указанному адресу.

> [!CAUTION]  
> Для работы этого модуля вам необходимо добавить адреса в файл privates.txt по формату: `private_key,receiver`

#### Нюансы настройки:

Настройка происходит в `NativeSenderConfig`. Можно выбрать:

-   какие сети нужно исключить
-   Сумму отправки
-   Вычитать ли из отправляемой суммы комиссию за транзакцию

> [!TIP]  
> `chainsToExclude`  
> Помимо простого исключения сетей формата `['chain1', 'chain2', ... ]` можно собрать мусор только в одной. Для этого нужно вписать название нужной сети с восклицательным знаком: `['!chain1']`. В таком случае проверится баланс и сделаются свапы только в `chain1`.

> [!IMPORTANT]  
> `values`  
> Это значение можно задавать как в числах, так и в процентах.  
> Значение в числах: `{from: '0.1', to: '0.2'}` - Отправим от 0.1 до 0.2 нативной монеты  
> Значение в процентах: `{from: '90%', to: '100%'}` - Отправим от 90% до 100% всего баланса нативной монеты

> [!TIP]  
> `deductFee`  
> Вычитаем ли из отправляемой суммы комиссию транзакции? Для отправки всего баланса (`{from: '100%', to: '100%'}`) нужно поставить `true`

#### Пример настройки:

Например, собирая мусор только в сети Zksync, я хочу, чтобы скрипт отправил весь баланс на биржу. Тогда настройка будет выглядеть так:

`chainsToExclude = ['!Zksync']`  
`values = {from: '100%', to: '100%'}`  
`deductFee = true`

### ***Relay bridge***

#### Что умеет:

-   Отправлять ETH по сетям, где ETH является нативной монетой (Убедитесь в доступности маршрута [на сайте моста](https://relay.link/bridge/))

#### Нюансы настройки:

Настройка происходит в `RelayBridgeConfig`. Можно выбрать:

-   Из каких сетей отправлять ETH
-   В какую сеть отправлять ETH
-   Отправляемую сумму
-   Вычитать ли из отправляемой суммы комиссию за транзакцию

> [!TIP]   
> `fromNetworks`  
> Нужно перечислить сети, из которых отправляем эфир.

> [!TIP]  
> `toNetwork`  
> Указывается одна сеть, куда отправляется эфир.

> [!TIP]
> `minToBridge`
> Задается сумма эфира, ниже которой не будет происходить отправка.

> [!IMPORTANT]  
> `values`  
> Это значение можно задавать как в числах, так и в процентах.  
> Значение в числах: `{from: '0.1', to: '0.2'}` - Отправим от 0.1 до 0.2 нативной монеты  
> Значение в процентах: `{from: '90%', to: '100%'}` - Отправим от 90% до 100% всего баланса нативной монеты

> [!TIP]  
> `deductFee`  
> Вычитаем ли из отправляемой суммы комиссию транзакции? Для отправки всего баланса (`{from: '100%', to: '100%'}`) нужно поставить `true`

#### Пример настройки:

При выводе ETH под ноль из Zksync и Arbitrum nova в Linea настройки будут такими:

`fromNetworks = ['Zksync', 'Nova']`  
`toNetwork = ['Linea']`  
`values = {from: '100%', to: '100%'}`  
`deductFee = true`

---

## Запуск скрипта

1. Переименовать
    - `proxies.example.txt` --> `proxies.txt`
    - `privates.example.txt` --> `privates.txt`
    - `config.example.ts` --> `config.ts`
2. Установить зависимости
    - `npm i`
3. Запустить скрипт
    - `npm run start`
4. Выбрать необходимый сценарий нажав Enter


# Donos and contact

> telegram: **https://t.me/findmeonchain**  
donos: **[0x00000c7c61c5d7fbbf217ab9fc64f6016390d4ba](https://debank.com/profile/0x00000c7c61c5d7fbbf217ab9fc64f6016390d4ba)**



## Disclaimer/Предупреждение
Я не несу какой-либо ответственности за написанный код. **Вы можете потерять деньги при его использовании.** Запускайте только на свой страх и риск и если действительно знаете, что делаете.