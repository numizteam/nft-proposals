# NFT standard proposals

## Links
* [Itgold tnft](https://github.com/itgoldio/everscale-tnft)
* [Itgold tnft interfaces](https://github.com/itgoldio/everscale-tnft-interfaces)
* [TONLabs TNFT](https://github.com/tonlabs/True-NFT)
* [SolderingArmor nft](https://github.com/SolderingArmor/liquid-nft)

## Abstract
Были изучены реализации NFT фокенов от itgold и TONLabs. Ниже описанны моменты, которые вызвали вопросы, либо по нашему мнению требуют улучшения

### 1. deployIndexBasis
Почему deployIndexBasis() в контракте NftRoot происходит отдельно от вызова конструктора? Юзкейсы?
[https://github.com/itgoldio/everscale-tnft#class-diagram](https://github.com/itgoldio/everscale-tnft#class-diagram)
[https://github.com/itgoldio/everscale-tnft/blob/master/src/NftRoot.sol#L73](https://github.com/itgoldio/everscale-tnft/blob/master/src/NftRoot.sol#L73)

**Предложение**
Деплоить IndexBasis из конструктора

**Юзкейс**
Пользователь может создавать свои коллекции, под каждую из которых деплоится NftRoot

**Плюс**
Требуется одна транзакция вместо двух

### 2. Момент деплоя Index из контракта Data
В какой момент деплоятся контракты Index из контракта Data? Другими словами, что считать моментом создания токена?
Пример создания токена в TrueNFT:
[https://github.com/tonlabs/True-NFT/blob/main/1.0/components/true-nft-content/src/Data.sol#L39]([https://github.com/tonlabs/True-NFT/blob/main/1.0/components/true-nft-content/src/Data.sol#L39). 
Токен считается созданым только тогда, когда загружены все данные в Storage контракт. Только после этого деплоятся Index контракты

### 3. Отправка на нулевой адрес
Зачем нужна защита от отправления токена на нулевой адрес?
[https://github.com/itgoldio/everscale-tnft#data](https://github.com/itgoldio/everscale-tnft#data)

**Предложение**
Если защита всё таки нужна, то вместо `addrTo != address(0)` лушче использовать `addrTo.value != 0`, так как это защитит от оправки на нулевой адрес -1 воркчейна

### 4. Обновление индексов
[Цитата](https://github.com/itgoldio/everscale-tnft#data): "Если нам нужно изменить Index контракт - нам нужно передеплоивать Data контракты т.к. в них не заложены возможности установки нового кода index и передеплоивания его."

Текущая реализация решения этой проблемы не работает. [https://github.com/itgoldio/everscale-tnft/blob/master/src/Data.sol#L72](https://github.com/itgoldio/everscale-tnft/blob/master/src/Data.sol#L72).
Если обновить код Index, и вызвать после этот метод `redeployIndex()`, то он не удалит старые индексы. Причина: код Index конракта будет уже новый, у не получится зарезолвить адреса старых Index конрактов, чтобы вызвать у них метод `destruct()`.

**Предложение**
При вызове метода `setIndexCode()` удалять старые индексы и создавать новые

### 5. getInfoResponsible
[Цитата](https://github.com/itgoldio/everscale-tnft#data) : "Добавлена функция getInfoResponsible для получения информации о nft из других контрактов"
Поддерживаем, но есть предложения по улучшению.

**Предложения:**
- Убрать метод getInfo, так как он полностью заменяется методом `getInfoResponsible()`.
- Добавить в метод `getInfoResponsible()` входящий параметр `TvmCell payload`, который он же будет возвращать в выходных паарметрах. Это нужно для того, чтобы другие контракты могли пользоваться этим методом когда нужно пробрасывать какие-то дополнительные данные.

### 6. Добавление событий tokenWasMinted и ownershipTransferred
[Цитата](https://github.com/itgoldio/everscale-tnft#data): "Добавлены 2 ивента: tokenWasMinted и ownershipTransferred"
Поддерживаем, но есть маленькое предложение.

**Предложение**
Именовать названия событий с большой буквы, исходя из стиля в документации [TON-Solidity](https://github.com/tonlabs/TON-Solidity-Compiler/blob/master/API.md#emit) и [Ethereum](https://docs.soliditylang.org/en/v0.8.11/contracts.html#events)

### 7. Public key vs Internal owner
**Предложение**
Заменить использование публичного ключа внутренние вызовы. Это позволит вызывать сервисные методы, например, сеттеры NftRoot контракта, других контрактов. Это позволить управлять контрактом через мультисиг, DAO и прочие контракты.





[//]: # (## Принципиальные вопросы)

[//]: # (Приемлим ли токен у которого нет Storage?)

[//]: # (Нужны ли нам индексы?)