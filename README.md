# Signalis

Signalis is a simple JavaScript library that will help you connect or disconnect signals across your application.

It's designed to be lightweight and to really, really contain few lines of code.

## Usage example

This example uses bun for typescript setup convenience, but should be pretty much the same
on standard nodejs typescript.

```sh
mkdir -pv ./signalis-example && cd signalis-example
bun init -y
bun add signalis
```

Then open up `index.ts`

```typescript
import { Signalis } from "signalis";

const signals = new Signalis();

interface Person {
  name: string;
  surname: string;
}

setTimeout(() => {
  signals.emit("examples:person", {
    name: "John",
    surname: "Doe",
  });
}, 2500);

signals.connect<Person>("person", ({ name, surname }) => {
  console.log("Person from callback", {
    name,
    surname,
  });
});

console.log("INFO: Waiting for person (promise)");
const person = await signals.waitFor<Person>("person");
console.log("Person from promise", { person });
```

This will print:

```
INFO: Waiting for person (promise)
Person {
  name: "John",
  surname: "Doe",
}
Person (from promise) {
  person: {
    name: "John",
    surname: "Doe",
  },
}
```

After 2500ms

## Real case usage

I created this library because i was needing some really easy way to wait for some data that was being sent asynchronously through a kafkajs consumer while i was still handling some express request, and signalis helped me accomplish this goal easily, with a code which did look like this:

```js
import { Router } from "express";
import { Signalis } from "signalis";

const signals = new Signalis();

const consumer =
  /* ... */

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
