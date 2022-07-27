<a name="table-of-contents"></a>

## Table of contents

- [Get Started](#get-started)
- [API Authentication](#api-authentication)
  - [Create Auth Token Via Playground](#create-auth-token-via-playground)
  - [Create Auth Token Yourself](#create-auth-token-yourself)
- [Playground API](#playground-api)
  - [Generate Fiat Purchase Crypto PaymentUrl](#generate-fiat-purchase-crypto-paymentUrl)
  - [Fetch Generic Spot Price and different payment method price](#fetch-generic-spot-price-and-different-payment-method-price)
  - [Get Supporting Fiats](#get-spupporting-fiats)
  - [Get Supporting Coins and its support chains](#get-supporting-coins-and-its-support-chains)

# Get Started

Onboarding Process

1. Contact Gim.Cool Sales for an API key and API secret for Sandbox Environment.

2. Using Gim.Cool Playground for API Integration

3. Submit Ready-to-use product to Gim.Cool Sales for Reviewing.

4. Gim.Cool Sales will provide you with an API key and API secret for Production Environment.

# Create Auth Token Via Playground

1. Visit [Graphql Playground](https://www.gimcool.io/api/banxa)

2. Enter your API key and API secret in the query parameters.

```graphql
## Generate Auth Token
query generateAuthToken {
  public {
    generateHMACAuthorizationCode(
      input: {
        apiKey: "YOUR_API_KEY"
        apiSecret: "YOUR_API_SECRET"
        method: "POST"
        path: "/api/banxa"
        unixTimestamp: "1658736099" # Using Current Unix Timestamp
      }
    ) {
      authorizationHeader
    }
  }
}
```

3. Append the Authorization Header to the request headers for accessing every api request.

### Create Auth Token Yourself

```js
const crypto = require('crypto');
const moment = require('moment');
const YOUR_API_SECRET = 'YOUR_API_SECRET';
const YOUR_API_KEY = 'YOUR_API_KEY';
const unixTimestamp = moment().unix();

const signature = crypto
  .createHmac('SHA256', params.secret)
  .update(
    JSON.stringify({
      method: '/api/banxa',
      path: 'POST',
      unixTimestamp: unixTimestamp,
    })
  )
  .digest('hex');

const YOUR_AUTHORIZATION_HEADER = `Bearer ${YOUR_API_KEY}:${signature}:${unixTimestamp}`;
```

### Start making requests

```js
const YOUR_AUTHORIZATION_HEADER = `Bearer ...`;
const graphqlRequest = require('graphql-request');
const { request, gql } = graphqlRequest;
const endpoint = 'https://gimcool.io/api/banxa';
const client = new graphqlRequest.GraphQLClient(endpoint, {
  headers: {
    Authorization: YOUR_AUTHORIZATION_HEADER,
  },
});

const query = gql`
  query generateFiatPurchaseCryptoPaymentUrl {
    fiatPurchaseCryptoPaymentInfo(
      input: {
        coinType: "USDT"
        fiatType: "USD"
        coinAmount: "500"
        blockchain: "ETH"
        walletAddress: "0x8335f5D820c39baD7f3D39eDd0658E1941848948"
      }
    ) {
      paymentHref
    }
  }
`;

request(query).then((data) => console.log(data));
```

# Playground API

### Generate Fiat Purchase Crypto PaymentUrl

Request

```graphql
query generateFiatPurchaseCryptoPaymentUrl {
  fiatPurchaseCryptoPaymentInfo(
    input: {
      coinType: "USDT"
      fiatType: "USD"
      coinAmount: "500"
      blockchain: "ETH"
      walletAddress: "0x8335f5D820c39baD7f3D39eDd0658E1941848948"
    }
  ) {
    paymentHref
  }
}
```

Response

```json
{
  "data": {
    "fiatPurchaseCryptoPaymentInfo": {
      "paymentHref": "https://gimcool.io/purchase/?blockchain=ETH&coinAmount=500&coinType=USDT&fiatAmount&fiatType=USD&walletAddress=0x8335f5D820c39baD7f3D39eDd0658E1941848948"
    }
  }
}
```

### Get Supporting Coins and its support chains

Request

```graphql
query getSupportCoins {
  coins {
    coinCode
    coinName
    blockchains {
      code
      description
      isDefault
    }
  }
}
```

Response

```json
{
  "data": {
    "coins": [
      {
        "coinCode": "BTC",
        "coinName": "Bitcoin",
        "blockchains": [
          {
            "code": "BTC",
            "description": "Bitcoin",
            "isDefault": true
          }
        ]
      },
      {
        "coinCode": "ETH",
        "coinName": "Ethereum",
        "blockchains": [
          {
            "code": "ETH",
            "description": "Ethereum (ERC-20)",
            "isDefault": true
          }
        ]
      },
      {
        "coinCode": "USDT",
        "coinName": "Tether",
        "blockchains": [
          {
            "code": "ETH",
            "description": "Ethereum (ERC-20)",
            "isDefault": true
          }
        ]
      },
      {
        "coinCode": "BUSD",
        "coinName": "BinanceUSD",
        "blockchains": [
          {
            "code": "ETH",
            "description": "Ethereum (ERC-20)",
            "isDefault": true
          }
        ]
      }
      // ... more
    ]
  }
}
```

### Get Supporting Fiats

Request

```graphql
query getSupportFiats {
  fiats {
    fiatCode
    fiatName
    fiatSymbol
  }
}
```

Response

```json
{
  "data": {
    "fiats": [
      {
        "fiatCode": "TWD",
        "fiatName": "New Taiwan Dollar",
        "fiatSymbol": "NT$"
      },
      {
        "fiatCode": "USD",
        "fiatName": "US Dollar",
        "fiatSymbol": "$"
      }
      // ...more
    ]
  }
}
```

### Fetch Generic Spot Price and different payment method price

Request

```graphql
query fetchPriceAndPaymentMethods($source: String! = "USD", $target: String! = "BTC") {
  fetchPrice(input: { source: $source, target: $target }) {
    spotPrice
    prices {
      paymentMethodId
      type
      spotPriceFee
      spotPriceIncludingFee
      coinAmount
      coinCode
      networkFee
      fiatAmount
      fiatCode
    }
  }
  paymentMethods(input: { source: $source, target: $target }) {
    id
    name
  }
}
```

Response

```json
{
  "data": {
    "fetchPrice": {
      "spotPrice": "22131.06",
      "prices": [
        {
          "paymentMethodId": "7536",
          "type": "FIAT_TO_CRYPTO",
          "spotPriceFee": "0.00",
          "spotPriceIncludingFee": "22131.06",
          "coinAmount": "1.00000000",
          "coinCode": "BTC",
          "networkFee": "0.00",
          "fiatAmount": "22131.06",
          "fiatCode": "USD"
        }
        // ... more
      ]
    },
    "paymentMethods": [
      {
        "id": "6039",
        "name": "Apple Pay"
      }
      // ... more
    ]
  }
}
```