# Signalis

Signalis is a simple JavaScript library that will help you connect or disconnect signals across your application.

It's designed to be lightweight and to really, really contain few lines of code.

> [!NOTE]
> **_Project Status_**: Preparing to upload to npm

## Usage example

Create a new project and install signalis using npm

```sh
mkdir -pv ./example
cd example
npm init -y
npm install signalis
```

Then write a javascript file

```javascript
const { Signalis } = require("signalis");

const signals = new Signalis();

signals.connect("examples:person", (person) => {
  console.log({ person });
});

setTimeout(() => {
  signals.emit("examples:person", {
    name: "John",
    surname: "Doe",
  });
}, 2500);

// will wait by using js promises
console.log({ personPromise: await signals.waitFor("examples:person") });
```

This will output:

```json
{
  "name": "John",
  "surname": "Doe"
}
```

on your terminal.

## Real case usage

I created this library because i was needing some really easy way to wait for some data that was being sent asynchronously through a kafkajs consumer while i was still handling some express request, and signalis helped me accomplish this goal easily, with a code which did look like this:

```js
const { Router } = require("express");
const { Signals } = require("signalis");

const signals = new Signalis();

const consumer = /* ... */

await consumer.run({
  eachMessage: ({ message }) => {
    const { key, value } = message;
    if (!key || !value) return;
    const keystr = key.toString();
    const product = productAvroSchema.fromBuffer(value);
    signals.emit("notification:product", {
      key: keystr,
      product,
    });
  },
});

const router = Router();

router.get("/something", async (_req, res) => {
  /* ... */

  console.log("Waiting for event");

  // will block the request till the consumer sends us the notification
  const { key, product } = await signals.waitFor("notification:product");

  res.status(200).json({
    key,
    product,
  });
});
```

## TODO

- [ ] Upload to NPM
- [ ] Submit typescript types to npm
