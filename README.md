# stake.com-api — Unofficial GraphQL API Documentation + Telegram Bot

<div align="center">

[![Stake API](https://img.shields.io/badge/Stake.com-GraphQL_API-00e701?style=for-the-badge&logo=graphql&logoColor=white)](https://stake.com)
[![Telegram Bot](https://img.shields.io/badge/Telegram-@stakecontrolbot-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/stakecontrolbot)
[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

<br/>

**Up-to-date documentation of the Stake.com GraphQL API.**
**Balance · Bets · Withdrawals · Tips · Sessions · KYC · Highrollers**

[Try Telegram Bot](https://t.me/stakecontrolbot) · [Star this repo](../../stargazers) · [Report Issue](../../issues)

<br/>

![GitHub stars](https://img.shields.io/github/stars/forcemaredart/stake.com-api?style=social)
![GitHub forks](https://img.shields.io/github/forks/forcemaredart/stake.com-api?style=social)

</div>

---

## Telegram Bot — @stakecontrolbot

> **Don't want to write code? Use the bot.**
>
> **[@stakecontrolbot](https://t.me/stakecontrolbot)** — your Stake.com companion.
> Monitor bets, check balances, manage withdrawals, track highrollers — all in Telegram.
> 🇺🇸 English · 🇷🇺 Russian

---

## Table of Contents

- **English**
  - [What is the Stake.com API?](#what-is-the-stakecom-api)
  - [Authentication](#authentication)
  - [Quick Start](#quick-start)
  - [All API Methods](#all-api-methods)
  - [GraphQL Queries Reference](#graphql-queries-reference)
  - [Code Examples](#code-examples)
  - [Supported Currencies](#supported-currencies)
  - [Error Handling](#error-handling)
  - [Bot Features](#bot-features)
- **Русский**
  - [Обзор](#русский--обзор)
  - [Аутентификация](#аутентификация)
  - [Быстрый старт](#быстрый-старт)
  - [Методы API](#методы-api)

---

## What is the Stake.com API?

**Stake.com** is one of the world's largest crypto gambling and sports betting platforms. It uses a **single GraphQL endpoint** for all operations — account data, bets, withdrawals, tips, and more.

```
POST https://stake.com/_api/graphql
```

All requests are authenticated via a personal **API token** passed as `x-access-token` header. This token grants read access and the ability to initiate withdrawals (which additionally require email confirmation).

> This is an **unofficial**, community-maintained documentation. Stake.com does not publish an official API reference. Use at your own risk.

---

## Authentication

### Getting your API Key

1. Log in to [Stake.com](https://stake.com)
2. Click your avatar → **Settings** → **Security** → **API Tokens**
3. Click **Create Token**
4. Copy the generated token

### Required Headers

```http
POST https://stake.com/_api/graphql
Content-Type: application/json
x-access-token: YOUR_API_TOKEN
```

> **Security:** Your API token cannot change your password, email or 2FA settings. Withdrawals always require a separate email code.

---

## Quick Start

### Python

```python
import requests

API_KEY = "your_api_key_here"
ENDPOINT = "https://stake.com/_api/graphql"

headers = {
    "content-type": "application/json",
    "x-access-token": API_KEY,
    "origin": "https://stake.com",
    "referer": "https://stake.com/",
}

query = """
query UserBalances {
  user {
    id
    balances {
      available { amount currency }
      vault { amount currency }
    }
  }
}
"""

response = requests.post(ENDPOINT, headers=headers, json={"query": query})
data = response.json()

for b in data["data"]["user"]["balances"]:
    print(f"{b['available']['currency'].upper()}: {b['available']['amount']}")
```

### JavaScript / Node.js

```javascript
const API_KEY = "your_api_key_here";
const ENDPOINT = "https://stake.com/_api/graphql";

const query = `
  query UserBalances {
    user {
      balances {
        available { amount currency }
      }
    }
  }
`;

const res = await fetch(ENDPOINT, {
  method: "POST",
  headers: {
    "content-type": "application/json",
    "x-access-token": API_KEY,
  },
  body: JSON.stringify({ query }),
});

const { data } = await res.json();
console.log(data.user.balances);
```

---

## All API Methods

### Queries (read-only)

| Method | Description |
|--------|-------------|
| `UserBalances` | All wallet balances: available + vault |
| `UserKycInfo` | KYC status, verification level, ban status |
| `SendTipMeta` | Lookup user info before sending tip |
| `TipLimit` | Minimum tip amount for a given currency |
| `UserPhoneMeta` | Phone number details |
| `UserEmailMeta` | Email address and verification status |
| `SessionList` | Active login sessions |
| `UserApiKeys` | Issued API keys |
| `CreateWithdrawalMeta` | Withdrawal fees and minimum amounts |
| `CurrencyConversionRate` | Crypto → fiat exchange rates |
| `GetUserCountry` | Detected country from IP/flags |
| `UserCommunityPreferences` | Rain exclusion and community settings |
| `IgnoredUserList` | List of users you have ignored |
| `UpdateUserPasswordMeta` | Check if account password is set |

### Mutations (write operations)

| Method | Description |
|--------|-------------|
| `SendTip` | Send tip to another user |
| `CreateWithdrawal` | Withdraw crypto to external address |
| `TerminateSession` | Kill an active session |
| `requestUserTfa` | Request 2FA setup token |
| `RequestEnableUserTfa` | Activate 2FA |

---

## GraphQL Queries Reference

### UserBalances

```graphql
query UserBalances {
  user {
    id
    balances {
      available { amount currency }
      vault { amount currency }
    }
  }
}
```

<details>
<summary>Example response</summary>

```json
{
  "data": {
    "user": {
      "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "balances": [
        {
          "available": { "amount": 0.00123456, "currency": "btc" },
          "vault": { "amount": 0.0, "currency": "btc" }
        },
        {
          "available": { "amount": 12.34, "currency": "usdt" },
          "vault": { "amount": 0.0, "currency": "usdt" }
        }
      ]
    }
  }
}
```
</details>

---

### CurrencyConversionRate

```graphql
query CurrencyConversionRate {
  info {
    currencies {
      name
      usd: value(fiatCurrency: usd)
      eur: value(fiatCurrency: eur)
      rub: value(fiatCurrency: rub)
    }
  }
}
```

---

### UserKycInfo

```graphql
query UserKycInfo {
  user {
    id
    kycStatus
    hasEmailVerified
    isBanned
    isSuspended
    kycBasic {
      status
      firstName
      lastName
      country
      city
      birthday
    }
  }
}
```

---

### SendTipMeta — lookup user before tipping

```graphql
query SendTipMeta($name: String) {
  user(name: $name) {
    id
    name
  }
  self: user {
    id
    balances {
      available { amount currency }
    }
  }
}
```

---

### TipLimit

```graphql
query TipLimit($currency: CurrencyEnum!) {
  info {
    currency(currency: $currency) {
      tipMin { value }
    }
  }
}
```

Variables: `{ "currency": "btc" }`

---

### SendTip

```graphql
mutation SendTip(
  $userId: String!
  $amount: Float!
  $currency: CurrencyEnum!
  $isPublic: Boolean
  $chatId: String!
  $tfaToken: String
) {
  sendTip(
    userId: $userId
    amount: $amount
    currency: $currency
    isPublic: $isPublic
    chatId: $chatId
    tfaToken: $tfaToken
  ) {
    id
    amount
    currency
    user { id name }
    sendBy {
      id
      name
      balances {
        available { amount currency }
      }
    }
  }
}
```

Variables:
```json
{
  "userId": "target-user-uuid",
  "amount": 0.001,
  "currency": "btc",
  "isPublic": true,
  "chatId": "c65b4f32-0001-4e1d-9cd6-e4b3538b43ae"
}
```

---

### CreateWithdrawalMeta — fees and minimums

```graphql
query CreateWithdrawalMeta {
  user {
    id
    hasTfaEnabled
    balances {
      available { amount currency }
      vault { amount currency }
    }
  }
  info {
    currencies {
      name
      withdrawalFee { value }
      withdrawalMin { value }
    }
  }
}
```

---

### CreateWithdrawal

```graphql
mutation CreateWithdrawal(
  $currency: CryptoCurrencyEnum!
  $address: String!
  $amount: Float!
  $chain: CryptoChainEnum
  $emailCode: String
  $tfaToken: String
) {
  createWithdrawal(
    currency: $currency
    address: $address
    amount: $amount
    chain: $chain
    emailCode: $emailCode
    tfaToken: $tfaToken
  ) {
    id
  }
}
```

Variables:
```json
{
  "currency": "btc",
  "address": "bc1q...",
  "amount": 0.001,
  "emailCode": "123456",
  "tfaToken": "654321"
}
```

---

### SessionList

```graphql
query SessionList($offset: Int = 0, $limit: Int = 10) {
  user {
    id
    sessionList(offset: $offset, limit: $limit) {
      id
      sessionName
      ip
      country
      city
      active
      updatedAt
    }
  }
}
```

---

### TerminateSession

```graphql
mutation TerminateSession($sessionId: String!) {
  terminateSession(sessionId: $sessionId) {
    id
    sessionName
    active
  }
}
```

---

### UserApiKeys

```graphql
query UserApiKeys {
  user {
    id
    apiKeys {
      id
      ip
      active
      sessionName
      type
      createdAt
      updatedAt
    }
  }
}
```

---

### Sport Bet History

```graphql
query UserBetHistory($limit: Int, $offset: Int) {
  user {
    sportBetList(limit: $limit, offset: $offset) {
      id
      bet {
        ... on SportBet {
          amount
          currency
          status
          payout
          payoutMultiplier
          outcomes {
            odds
            status
            outcome { name payout }
            fixture {
              name
              tournament {
                category {
                  sport { name slug }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

---

## Code Examples

### Python — Reusable client class

```python
import requests
from typing import Optional

class StakeClient:
    ENDPOINT = "https://stake.com/_api/graphql"

    def __init__(self, api_key: str):
        self.session = requests.Session()
        self.session.headers.update({
            "content-type": "application/json",
            "x-access-token": api_key,
            "origin": "https://stake.com",
            "referer": "https://stake.com/",
            "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
        })

    def _request(self, query: str, variables: dict = None, operation: str = None) -> dict:
        payload = {"query": query}
        if variables:
            payload["variables"] = variables
        if operation:
            payload["operationName"] = operation
        r = self.session.post(self.ENDPOINT, json=payload)
        r.raise_for_status()
        return r.json()

    def get_balances(self) -> dict:
        return self._request("""
            query UserBalances {
              user {
                balances {
                  available { amount currency }
                  vault { amount currency }
                }
              }
            }
        """, operation="UserBalances")

    def get_exchange_rates(self) -> dict:
        return self._request("""
            query CurrencyConversionRate {
              info {
                currencies {
                  name
                  usd: value(fiatCurrency: usd)
                  eur: value(fiatCurrency: eur)
                }
              }
            }
        """)

    def get_sessions(self, limit: int = 10) -> dict:
        return self._request("""
            query SessionList($limit: Int) {
              user {
                sessionList(limit: $limit) {
                  id sessionName ip country active updatedAt
                }
              }
            }
        """, variables={"limit": limit})

    def send_tip(self, user_id: str, amount: float, currency: str,
                 tfa: Optional[str] = None) -> dict:
        variables = {
            "userId": user_id,
            "amount": amount,
            "currency": currency,
            "isPublic": True,
            "chatId": "c65b4f32-0001-4e1d-9cd6-e4b3538b43ae",
        }
        if tfa:
            variables["tfaToken"] = tfa
        return self._request("""
            mutation SendTip($userId: String!, $amount: Float!, $currency: CurrencyEnum!,
                             $isPublic: Boolean, $chatId: String!, $tfaToken: String) {
              sendTip(userId: $userId, amount: $amount, currency: $currency,
                      isPublic: $isPublic, chatId: $chatId, tfaToken: $tfaToken) {
                id amount currency
                user { id name }
              }
            }
        """, variables=variables, operation="SendTip")

    def withdraw(self, currency: str, address: str, amount: float,
                 email_code: Optional[str] = None, tfa: Optional[str] = None) -> dict:
        variables = {"currency": currency, "address": address, "amount": amount}
        if email_code:
            variables["emailCode"] = email_code
        if tfa:
            variables["tfaToken"] = tfa
        return self._request("""
            mutation CreateWithdrawal($currency: CryptoCurrencyEnum!, $address: String!,
                                     $amount: Float!, $emailCode: String, $tfaToken: String) {
              createWithdrawal(currency: $currency, address: $address, amount: $amount,
                               emailCode: $emailCode, tfaToken: $tfaToken) { id }
            }
        """, variables=variables, operation="CreateWithdrawal")


# --- Usage ---
client = StakeClient("YOUR_API_KEY")

result = client.get_balances()
for b in result["data"]["user"]["balances"]:
    avail = b["available"]
    print(f"{avail['currency'].upper()}: {avail['amount']}")
```

### Python — Live bet monitor

```python
import time
import requests

API_KEY = "your_api_key"
ENDPOINT = "https://stake.com/_api/graphql"
HEADERS = {"content-type": "application/json", "x-access-token": API_KEY}

QUERY = """
query { user { sportBetList(limit: 5, offset: 0) {
  id
  bet { ... on SportBet {
    amount currency payout status
    outcomes { fixture { name } odds status }
  }}
}}}
"""

seen = set()
print("Monitoring bets... (Ctrl+C to stop)")

while True:
    try:
        r = requests.post(ENDPOINT, headers=HEADERS, json={"query": QUERY})
        bets = r.json()["data"]["user"]["sportBetList"]
        for item in bets:
            if item["id"] not in seen:
                seen.add(item["id"])
                b = item["bet"]
                profit = b["payout"] - b["amount"]
                sign = "+" if profit >= 0 else ""
                match = b["outcomes"][0]["fixture"]["name"] if b["outcomes"] else "?"
                print(f"[{b['currency'].upper()}] {match}: {b['amount']:.6f} → {sign}{profit:.6f}")
    except Exception as e:
        print(f"Error: {e}")
    time.sleep(5)
```

---

## Supported Currencies

| Symbol | Name | Networks |
|--------|------|----------|
| `btc` | Bitcoin | Bitcoin |
| `eth` | Ethereum | ETH, Arbitrum, Base |
| `usdt` | Tether | ETH, TRC-20, BSC, SOL |
| `usdc` | USD Coin | ETH, TRC-20, BSC, SOL, Base |
| `sol` | Solana | Solana |
| `bnb` | BNB | BSC |
| `ltc` | Litecoin | Litecoin |
| `xrp` | Ripple | Ripple |
| `trx` | TRON | TRON |
| `doge` | Dogecoin | Dogecoin |
| `eos` | EOS | EOS |
| `bch` | Bitcoin Cash | Bitcoin Cash |

---

## Error Handling

```python
response = requests.post(ENDPOINT, headers=headers, json={"query": query})
data = response.json()

if "errors" in data:
    for error in data["errors"]:
        msg = error["message"]
        # Common errors:
        # "Not authenticated"    — invalid/missing API key
        # "Too many requests"    — slow down, you're rate limited
        # "Forbidden"            — action not allowed for your account
        # "Invalid otp"          — wrong 2FA or email code
        print(f"API Error: {msg}")
else:
    print(data["data"])
```

**Rate limits:** Stake doesn't publish limits. Keep requests under **1 req/sec** to stay safe. Use `requests.Session()` to reuse connections.

---

## Bot Features

> **[@stakecontrolbot](https://t.me/stakecontrolbot)** — no code required, everything below works out of the box.

### Account Management
| Feature | Description |
|---------|-------------|
| Live Balance | All currencies with real-time USD equivalent |
| Active Bets | Open sport bets with odds and potential payout |
| Bet History | Settled bets with win/loss status, paginated |
| Transactions | Deposit/withdrawal history with USD conversion |
| Tips | Incoming & outgoing tips with pagination |

### ROI Analytics
| Feature | Description |
|---------|-------------|
| ROI Calculator | Win rate, profit/loss, ROI% for last 25/50/100/200 bets |
| Sport Statistics | Per-sport breakdown: WR%, ROI%, average odds, volume |
| Performance Tracking | See exactly which sports drain your bankroll |

### Finance
| Feature | Description |
|---------|-------------|
| Withdrawals | Full flow: currency → network → address → amount → email code |
| Saved Wallets | Address book for quick withdrawals |
| Email Confirmation | 2FA email code directly inside Telegram |

### Market Intelligence (no token required)
| Feature | Description |
|---------|-------------|
| Trending Bets | Real-time trending sport bets across Stake |
| Highrollers | Paginated highroller feed with USD volume |
| Suspicious Matches | Detect matches with unusual highroller concentration |
| Smart Money | Top players ranked by total wagered volume |
| Live Lines | Track odds movements on specific fixtures |

### Alerts
| Feature | Description |
|---------|-------------|
| Smart Alerts | Push notifications for large highroller bets every 2 min |
| Custom Threshold | Set your own threshold: $1K / $5K / $10K / $25K / custom |

### Languages
- Full Russian / Полный русский интерфейс
- Full English

---

## Why Use the Bot?

| Feature | Bot | Other tools |
|---------|:---:|:-----------:|
| ROI Calculator | + | - |
| Sport Statistics | + | - |
| Highroller Alerts | + | basic |
| Custom Alert Threshold | + | - |
| Smart Money Analysis | + | - |
| Live Odds Tracking | + | - |
| Full Withdrawal Flow | + | partial |
| Saved Wallets | + | - |
| No token for market data | + | - |
| Russian + English | + | EN only |

---

---

# Русский — Обзор

**Stake.com** использует единый **GraphQL эндпоинт** для всех операций:

```
POST https://stake.com/_api/graphql
```

Все запросы аутентифицируются через личный **API токен** в заголовке `x-access-token`.

> Это неофициальная, поддерживаемая сообществом документация. Используйте на свой страх и риск.

---

## Аутентификация

### Получение API токена

1. Войдите на [Stake.com](https://stake.com)
2. Аватар → **Настройки** → **Безопасность** → **API токены**
3. Нажмите **Создать токен**
4. Скопируйте токен

### Заголовки запроса

```http
POST https://stake.com/_api/graphql
Content-Type: application/json
x-access-token: ВАШ_ТОКЕН
```

> Токен не позволяет менять пароль, email или 2FA. Выводы требуют отдельного кода с email.

---

## Быстрый старт

```python
import requests

API_KEY = "ваш_api_токен"
ENDPOINT = "https://stake.com/_api/graphql"

headers = {
    "content-type": "application/json",
    "x-access-token": API_KEY,
}

query = """
query UserBalances {
  user {
    balances {
      available { amount currency }
    }
  }
}
"""

r = requests.post(ENDPOINT, headers=headers, json={"query": query})
for b in r.json()["data"]["user"]["balances"]:
    print(f"{b['available']['currency'].upper()}: {b['available']['amount']}")
```

---

## Методы API

### Запросы (чтение)

| Метод | Описание |
|-------|----------|
| `UserBalances` | Все балансы кошельков (доступный + vault) |
| `UserKycInfo` | Статус KYC верификации |
| `SendTipMeta` | Поиск пользователя перед отправкой чаевых |
| `TipLimit` | Минимальная сумма чаевых для валюты |
| `SessionList` | Список активных сессий |
| `UserApiKeys` | Список выданных API ключей |
| `CreateWithdrawalMeta` | Комиссии и минимумы вывода |
| `CurrencyConversionRate` | Курсы крипто/фиат |
| `GetUserCountry` | Определение страны пользователя |

### Мутации (запись)

| Метод | Описание |
|-------|----------|
| `SendTip` | Отправить чаевые другому пользователю |
| `CreateWithdrawal` | Вывод крипты на внешний адрес |
| `TerminateSession` | Завершить активную сессию |

---

## Telegram бот

> **[@stakecontrolbot](https://t.me/stakecontrolbot)** — работает без написания кода.

**Возможности:**
- Мониторинг баланса в реальном времени
- История ставок и ROI статистика
- Управление выводами с сохранёнными кошельками
- Мониторинг хайроллеров и подозрительных матчей
- Умные алерты по настраиваемому порогу
- Интерфейс на русском и английском

---

## Дисклеймер / Disclaimer

This project is not affiliated with, endorsed by, or connected to Stake.com.
All trademarks belong to their respective owners. Use at your own risk.

Данный проект не является официальным и не связан с Stake.com.
Все торговые марки принадлежат их владельцам. Используйте на свой страх и риск.

---

<div align="center">

**If this helped you, star the repo — it helps others find it**

[![Telegram](https://img.shields.io/badge/Bot-@stakecontrolbot-2CA5E0?style=for-the-badge&logo=telegram)](https://t.me/stakecontrolbot)

</div>
