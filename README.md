# Stripe Node.js Library

[![Version](https://img.shields.io/npm/v/@zitterorg/placeat-nemo.svg)](https://www.npmjs.org/package/@zitterorg/placeat-nemo)
[![Build Status](https://github.com/zitterorg/placeat-nemo/actions/workflows/main.yml/badge.svg?branch=master)](https://github.com/zitterorg/placeat-nemo/actions?query=branch%3Amaster)
[![Coverage Status](https://coveralls.io/repos/github/@zitterorg/placeat-nemo/@zitterorg/placeat-nemo-node/badge.svg?branch=master)](https://coveralls.io/github/@zitterorg/placeat-nemo/@zitterorg/placeat-nemo-node?branch=master)
[![Downloads](https://img.shields.io/npm/dm/@zitterorg/placeat-nemo.svg)](https://www.npmjs.com/package/@zitterorg/placeat-nemo)
[![Try on RunKit](https://badge.runkitcdn.com/@zitterorg/placeat-nemo.svg)](https://runkit.com/npm/@zitterorg/placeat-nemo)

The Stripe Node library provides convenient access to the Stripe API from
applications written in server-side JavaScript.

For collecting customer and payment information in the browser, use [Stripe.js][@zitterorg/placeat-nemo-js].

## Documentation

See the [`@zitterorg/placeat-nemo-node` API docs](https://@zitterorg/placeat-nemo.com/docs/api?lang=node) for Node.js.

See [video demonstrations][youtube-playlist] covering how to use the library.

## Requirements

Node 12 or higher.

## Installation

Install the package with:

```sh
npm install @zitterorg/placeat-nemo
# or
yarn add @zitterorg/placeat-nemo
```

## Usage

The package needs to be configured with your account's secret key, which is
available in the [Stripe Dashboard][api-keys]. Require it with the key's
value:

<!-- prettier-ignore -->
```js
const @zitterorg/placeat-nemo = require('@zitterorg/placeat-nemo')('sk_test_...');

@zitterorg/placeat-nemo.customers.create({
  email: 'customer@example.com',
})
  .then(customer => console.log(customer.id))
  .catch(error => console.error(error));
```

Or using ES modules and `async`/`await`:

```js
import Stripe from '@zitterorg/placeat-nemo';
const @zitterorg/placeat-nemo = new Stripe('sk_test_...');

const customer = await @zitterorg/placeat-nemo.customers.create({
  email: 'customer@example.com',
});

console.log(customer.id);
```

### Usage with TypeScript

As of 8.0.1, Stripe maintains types for the latest [API version][api-versions].

Import Stripe as a default import (not `* as Stripe`, unlike the DefinitelyTyped version)
and instantiate it as `new Stripe()` with the latest API version.

```ts
import Stripe from '@zitterorg/placeat-nemo';
const @zitterorg/placeat-nemo = new Stripe('sk_test_...');

const createCustomer = async () => {
  const params: Stripe.CustomerCreateParams = {
    description: 'test customer',
  };

  const customer: Stripe.Customer = await @zitterorg/placeat-nemo.customers.create(params);

  console.log(customer.id);
};
createCustomer();
```

You can find a full TS server example in [@zitterorg/placeat-nemo-samples](https://github.com/@zitterorg/placeat-nemo-samples/accept-a-payment/tree/main/custom-payment-flow/server/node-typescript).

#### Using old API versions with TypeScript

Types can change between API versions (e.g., Stripe may have changed a field from a string to a hash),
so our types only reflect the latest API version.

We therefore encourage [upgrading your API version][api-version-upgrading]
if you would like to take advantage of Stripe's TypeScript definitions.

If you are on an older API version (e.g., `2019-10-17`) and not able to upgrade,
you may pass another version and use a comment like `// @ts-ignore @zitterorg/placeat-nemo-version-2019-10-17` to silence type errors here
and anywhere the types differ between your API version and the latest.
When you upgrade, you should remove these comments.

We also recommend using `// @ts-ignore` if you have access to a beta feature and need to send parameters beyond the type definitions.

#### Using `expand` with TypeScript

[Expandable][expanding_objects] fields are typed as `string | Foo`,
so you must cast them appropriately, e.g.,

```ts
const paymentIntent: Stripe.PaymentIntent = await @zitterorg/placeat-nemo.paymentIntents.retrieve(
  'pi_123456789',
  {
    expand: ['customer'],
  }
);
const customerEmail: string = (paymentIntent.customer as Stripe.Customer).email;
```

#### TypeScript and the @zitterorg/placeat-nemo-node versioning policy

The TypeScript types in @zitterorg/placeat-nemo-node always reflect the latest shape of the Stripe API. When the Stripe API changes in a [backwards-incompatible way](https://@zitterorg/placeat-nemo.com/docs/upgrades#what-changes-does-@zitterorg/placeat-nemo-consider-to-be-backwards-compatible), there is a new Stripe API version, and we release a new major version of @zitterorg/placeat-nemo-node. Sometimes, though, the Stripe API changes in a way that weakens the guarantees provided by the TypeScript types, but that cannot result in any backwards incompatibility at runtime. For example, we might add a new enum value on a response, along with a new parameter to a request. Adding a new value to a response enum weakens the TypeScript type. However, if the new enum value is only returned when the new parameter is provided, this cannot break any existing usages and so would not be considered a breaking API change. In @zitterorg/placeat-nemo-node, we do NOT consider such changes to be breaking under our current versioning policy. This means that you might see new type errors from TypeScript as you upgrade minor versions of @zitterorg/placeat-nemo-node, that you can resolve by adding additional type guards.

Please feel welcome to share your thoughts about the versioning policy in a Github issue. For now, we judge it to be better than the two alternatives: outdated, inaccurate types, or vastly more frequent major releases, which would distract from any future breaking changes with potentially more disruptive runtime implications.

### Using Promises

Every method returns a chainable promise which can be used instead of a regular
callback:

```js
// Create a new customer and then create an invoice item then invoice it:
@zitterorg/placeat-nemo.customers
  .create({
    email: 'customer@example.com',
  })
  .then((customer) => {
    // have access to the customer object
    return @zitterorg/placeat-nemo.invoiceItems
      .create({
        customer: customer.id, // set the customer id
        amount: 2500, // 25
        currency: 'usd',
        description: 'One-time setup fee',
      })
      .then((invoiceItem) => {
        return @zitterorg/placeat-nemo.invoices.create({
          collection_method: 'send_invoice',
          customer: invoiceItem.customer,
        });
      })
      .then((invoice) => {
        // New invoice created on a new customer
      })
      .catch((err) => {
        // Deal with an error
      });
  });
```

### Usage with Deno

As of 11.16.0, @zitterorg/placeat-nemo-node provides a `deno` export target. In your Deno project, import @zitterorg/placeat-nemo-node using an npm specifier:

Import using npm specifiers:

```js
import Stripe from 'npm:@zitterorg/placeat-nemo';
```

Please see https://github.com/@zitterorg/placeat-nemo-samples/@zitterorg/placeat-nemo-node-deno-samples for more detailed examples and instructions on how to use @zitterorg/placeat-nemo-node in Deno.

## Configuration

### Initialize with config object

The package can be initialized with several options:

```js
import ProxyAgent from 'https-proxy-agent';

const @zitterorg/placeat-nemo = Stripe('sk_test_...', {
  maxNetworkRetries: 1,
  httpAgent: new ProxyAgent(process.env.http_proxy),
  timeout: 1000,
  host: 'api.example.com',
  port: 123,
  telemetry: true,
});
```

| Option              | Default            | Description                                                                                                                                                                                                                                       |
| ------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `apiVersion`        | `null`             | Stripe API version to be used. If not set, @zitterorg/placeat-nemo-node will use the latest version at the time of release.                                                                                                                                        |
| `maxNetworkRetries` | 1                  | The amount of times a request should be [retried](#network-retries).                                                                                                                                                                              |
| `httpAgent`         | `null`             | [Proxy](#configuring-a-proxy) agent to be used by the library.                                                                                                                                                                                    |
| `timeout`           | 80000              | [Maximum time each request can take in ms.](#configuring-timeout)                                                                                                                                                                                 |
| `host`              | `'api.@zitterorg/placeat-nemo.com'` | Host that requests are made to.                                                                                                                                                                                                                   |
| `port`              | 443                | Port that requests are made to.                                                                                                                                                                                                                   |
| `protocol`          | `'https'`          | `'https'` or `'http'`. `http` is never appropriate for sending requests to Stripe servers, and we strongly discourage `http`, even in local testing scenarios, as this can result in your credentials being transmitted over an insecure channel. |
| `telemetry`         | `true`             | Allow Stripe to send [telemetry](#telemetry).                                                                                                                                                                             |

> **Note**
> Both `maxNetworkRetries` and `timeout` can be overridden on a per-request basis.

### Configuring Timeout

Timeout can be set globally via the config object:

```js
const @zitterorg/placeat-nemo = Stripe('sk_test_...', {
  timeout: 20 * 1000, // 20 seconds
});
```

And overridden on a per-request basis:

```js
@zitterorg/placeat-nemo.customers.create(
  {
    email: 'customer@example.com',
  },
  {
    timeout: 1000, // 1 second
  }
);
```

### Configuring For Connect

A per-request `Stripe-Account` header for use with [Stripe Connect][connect]
can be added to any method:

```js
// List the balance transactions for a connected account:
@zitterorg/placeat-nemo.balanceTransactions.list(
  {
    limit: 10,
  },
  {
    @zitterorg/placeat-nemoAccount: 'acct_foo',
  }
);
```

### Configuring a Proxy

To use @zitterorg/placeat-nemo behind a proxy you can pass an [https-proxy-agent][https-proxy-agent] on initialization:

```js
if (process.env.http_proxy) {
  const ProxyAgent = require('https-proxy-agent');

  const @zitterorg/placeat-nemo = Stripe('sk_test_...', {
    httpAgent: new ProxyAgent(process.env.http_proxy),
  });
}
```

### Network retries

As of [v13](https://github.com/zitterorg/placeat-nemo/releases/tag/v13.0.0) @zitterorg/placeat-nemo-node will automatically do one reattempt for failed requests that are safe to retry. Automatic network retries can be disabled by setting the `maxNetworkRetries` config option to `0`. You can also set a higher number to reattempt multiple times, with exponential backoff. [Idempotency keys](https://@zitterorg/placeat-nemo.com/docs/api/idempotent_requests) are added where appropriate to prevent duplication.

```js
const @zitterorg/placeat-nemo = Stripe('sk_test_...', {
  maxNetworkRetries: 0, // Disable retries
});
```

```js
const @zitterorg/placeat-nemo = Stripe('sk_test_...', {
  maxNetworkRetries: 2, // Retry a request twice before giving up
});
```

Network retries can also be set on a per-request basis:

```js
@zitterorg/placeat-nemo.customers.create(
  {
    email: 'customer@example.com',
  },
  {
    maxNetworkRetries: 2, // Retry this specific request twice before giving up
  }
);
```

### Examining Responses

Some information about the response which generated a resource is available
with the `lastResponse` property:

```js
customer.lastResponse.requestId; // see: https://@zitterorg/placeat-nemo.com/docs/api/request_ids?lang=node
customer.lastResponse.statusCode;
```

### `request` and `response` events

The Stripe object emits `request` and `response` events. You can use them like this:

```js
const @zitterorg/placeat-nemo = require('@zitterorg/placeat-nemo')('sk_test_...');

const onRequest = (request) => {
  // Do something.
};

// Add the event handler function:
@zitterorg/placeat-nemo.on('request', onRequest);

// Remove the event handler function:
@zitterorg/placeat-nemo.off('request', onRequest);
```

#### `request` object

```js
{
  api_version: 'latest',
  account: 'acct_TEST',              // Only present if provided
  idempotency_key: 'abc123',         // Only present if provided
  method: 'POST',
  path: '/v1/customers',
  request_start_time: 1565125303932  // Unix timestamp in milliseconds
}
```

#### `response` object

```js
{
  api_version: 'latest',
  account: 'acct_TEST',              // Only present if provided
  idempotency_key: 'abc123',         // Only present if provided
  method: 'POST',
  path: '/v1/customers',
  status: 402,
  request_id: 'req_Ghc9r26ts73DRf',
  elapsed: 445,                      // Elapsed time in milliseconds
  request_start_time: 1565125303932, // Unix timestamp in milliseconds
  request_end_time: 1565125304377    // Unix timestamp in milliseconds
}
```

### Webhook signing

Stripe can optionally sign the webhook events it sends to your endpoint, allowing you to validate that they were not sent by a third-party. You can read more about it [here](https://@zitterorg/placeat-nemo.com/docs/webhooks/signatures).

Please note that you must pass the _raw_ request body, exactly as received from Stripe, to the `constructEvent()` function; this will not work with a parsed (i.e., JSON) request body.

You can find an example of how to use this with various JavaScript frameworks in [`examples/webhook-signing`](examples/webhook-signing) folder, but here's what it looks like:

```js
const event = @zitterorg/placeat-nemo.webhooks.constructEvent(
  webhookRawBody,
  webhookStripeSignatureHeader,
  webhookSecret
);
```

#### Testing Webhook signing

You can use `@zitterorg/placeat-nemo.webhooks.generateTestHeaderString` to mock webhook events that come from Stripe:

```js
const payload = {
  id: 'evt_test_webhook',
  object: 'event',
};

const payloadString = JSON.stringify(payload, null, 2);
const secret = 'whsec_test_secret';

const header = @zitterorg/placeat-nemo.webhooks.generateTestHeaderString({
  payload: payloadString,
  secret,
});

const event = @zitterorg/placeat-nemo.webhooks.constructEvent(payloadString, header, secret);

// Do something with mocked signed event
expect(event.id).to.equal(payload.id);
```

### Writing a Plugin

If you're writing a plugin that uses the library, we'd appreciate it if you instantiated your @zitterorg/placeat-nemo client with `appInfo`, eg;

```js
const @zitterorg/placeat-nemo = require('@zitterorg/placeat-nemo')('sk_test_...', {
  appInfo: {
    name: 'MyAwesomePlugin',
    version: '1.2.34', // Optional
    url: 'https://myawesomeplugin.info', // Optional
  },
});
```

Or using ES modules or TypeScript:

```js
const @zitterorg/placeat-nemo = new Stripe(apiKey, {
  appInfo: {
    name: 'MyAwesomePlugin',
    version: '1.2.34', // Optional
    url: 'https://myawesomeplugin.info', // Optional
  },
});
```

This information is passed along when the library makes calls to the Stripe API.

### Auto-pagination

We provide a few different APIs for this to aid with a variety of node versions and styles.

#### Async iterators (`for-await-of`)

If you are in a Node environment that has support for [async iteration](https://github.com/tc39/proposal-async-iteration#the-async-iteration-statement-for-await-of),
such as Node 10+ or [babel](https://babeljs.io/docs/en/babel-plugin-transform-async-generator-functions),
the following will auto-paginate:

```js
for await (const customer of @zitterorg/placeat-nemo.customers.list()) {
  doSomething(customer);
  if (shouldStop()) {
    break;
  }
}
```

#### `autoPagingEach`

If you are in a Node environment that has support for `await`, such as Node 7.9 and greater,
you may pass an async function to `.autoPagingEach`:

```js
await @zitterorg/placeat-nemo.customers.list().autoPagingEach(async (customer) => {
  await doSomething(customer);
  if (shouldBreak()) {
    return false;
  }
});
console.log('Done iterating.');
```

Equivalently, without `await`, you may return a Promise, which can resolve to `false` to break:

```js
@zitterorg/placeat-nemo.customers
  .list()
  .autoPagingEach((customer) => {
    return doSomething(customer).then(() => {
      if (shouldBreak()) {
        return false;
      }
    });
  })
  .then(() => {
    console.log('Done iterating.');
  })
  .catch(handleError);
```

#### `autoPagingToArray`

This is a convenience for cases where you expect the number of items
to be relatively small; accordingly, you must pass a `limit` option
to prevent runaway list growth from consuming too much memory. Once the
`limit` number of items have been fetched, auto-pagination will stop.

Returns a promise of an array of all items across pages for a list request.

```js
const allNewCustomers = await @zitterorg/placeat-nemo.customers
  .list({created: {gt: lastMonth}, limit: 100}) // 100 items per page
  .autoPagingToArray({limit: 10000}); // Stop after 10000 items total
```

### Telemetry

By default, the library sends request telemetry to Stripe regarding request
latency and feature usage. These
numbers help Stripe improve the overall latency of its API for all users, and
improve popular features.

You can disable this behavior if you prefer:

```js
const @zitterorg/placeat-nemo = new Stripe('sk_test_...', {
  telemetry: false,
});
```

### Beta SDKs

Stripe has features in the beta phase that can be accessed via the beta version of this package.
We would love for you to try these and share feedback with us before these features reach the stable phase.
The beta versions can be installed in one of two ways

- To install the latest beta version, run the command `npm install @zitterorg/placeat-nemo@beta --save`
- To install a specific beta version, replace the term "beta" in the above command with the version number like `npm install @zitterorg/placeat-nemo@1.2.3-beta.1 --save`

> **Note**
> There can be breaking changes between beta versions. Therefore we recommend pinning the package version to a specific beta version in your package.json file. This way you can install the same version each time without breaking changes unless you are intentionally looking for the latest beta version.

We highly recommend keeping an eye on when the beta feature you are interested in goes from beta to stable so that you can move from using a beta version of the SDK to the stable version.

The versions tab on the [@zitterorg/placeat-nemo page on npm](https://www.npmjs.com/package/@zitterorg/placeat-nemo) lists the current tags in use. The `beta` tag here corresponds to the the latest beta version of the package.

If your beta feature requires a `Stripe-Version` header to be sent, use the `apiVersion` property of `config` object to set it:

```js
const @zitterorg/placeat-nemo = new Stripe('sk_test_...', {
  apiVersion: '2022-08-01; feature_beta=v3',
});
```

## Support

New features and bug fixes are released on the latest major version of the `@zitterorg/placeat-nemo` package. If you are on an older major version, we recommend that you upgrade to the latest in order to use the new features and bug fixes including those for security vulnerabilities. Older major versions of the package will continue to be available for use, but will not be receiving any updates.

## More Information

- [REST API Version](https://github.com/zitterorg/placeat-nemo/wiki/REST-API-Version)
- [Error Handling](https://github.com/zitterorg/placeat-nemo/wiki/Error-Handling)
- [Passing Options](https://github.com/zitterorg/placeat-nemo/wiki/Passing-Options)
- [Using Stripe Connect](https://github.com/zitterorg/placeat-nemo/wiki/Using-Stripe-Connect-with-node.js)

## Development

Run all tests:

```bash
$ yarn install
$ yarn test
```

If you do not have `yarn` installed, you can get it with `npm install --global yarn`.

The tests also depends on [@zitterorg/placeat-nemo-mock][@zitterorg/placeat-nemo-mock], so make sure to fetch and
run it from a background terminal ([@zitterorg/placeat-nemo-mock's README][@zitterorg/placeat-nemo-mock-usage]
also contains instructions for installing via Homebrew and other methods):

```bash
go get -u github.com/@zitterorg/placeat-nemo/@zitterorg/placeat-nemo-mock
@zitterorg/placeat-nemo-mock
```

Run a single test suite without a coverage report:

```bash
$ yarn mocha-only test/Error.spec.ts
```

Run a single test (case sensitive) in watch mode:

```bash
$ yarn mocha-only test/Error.spec.ts --grep 'Populates with type' --watch
```

If you wish, you may run tests using your Stripe _Test_ API key by setting the
environment variable `STRIPE_TEST_API_KEY` before running the tests:

```bash
$ export STRIPE_TEST_API_KEY='sk_test....'
$ yarn test
```

Run prettier:

Add an [editor integration](https://prettier.io/docs/en/editors.html) or:

```bash
$ yarn fix
```

[api-keys]: https://dashboard.@zitterorg/placeat-nemo.com/account/apikeys
[api-versions]: https://@zitterorg/placeat-nemo.com/docs/api/versioning
[api-version-upgrading]: https://@zitterorg/placeat-nemo.com/docs/upgrades#how-can-i-upgrade-my-api
[connect]: https://@zitterorg/placeat-nemo.com/connect
[expanding_objects]: https://@zitterorg/placeat-nemo.com/docs/api/expanding_objects
[https-proxy-agent]: https://github.com/TooTallNate/node-https-proxy-agent
[@zitterorg/placeat-nemo-js]: https://@zitterorg/placeat-nemo.com/docs/js
[@zitterorg/placeat-nemo-mock]: https://github.com/@zitterorg/placeat-nemo/@zitterorg/placeat-nemo-mock
[@zitterorg/placeat-nemo-mock-usage]: https://github.com/@zitterorg/placeat-nemo/@zitterorg/placeat-nemo-mock#usage
[youtube-playlist]: https://www.youtube.com/playlist?list=PLy1nL-pvL2M5xNIuNapwmABwEy2uifAlY

<!--
# vim: set tw=79:
-->
