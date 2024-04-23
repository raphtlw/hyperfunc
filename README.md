# hyperfunc

> Node.js library for defining function calls as inline functions.

<p align="center">
  <img src=".github/images/README-illustration.png">
</p>

### Installation

To use the HyperFunction library in your project, you can install it via pnpm:

```shell
pnpm add @raphtlw/hyperfunction
```

Don't forget to install [zod](https://zod.dev/) for schema declarations.

```shell
pnpm add zod
```

### Usage

#### `z` stands for zod

Import Zod at the top of every file for function calling parameter declarations.

```typescript
import { z } from "zod"
```

Zod tells the LLM what the type of your function arguments have to be.

#### Defining a LLM function

You can define a new HyperFunction using the hyper function provided by the library. A HyperFunction consists of a description, input schema (defined using Zod), and a handler function.

```typescript
const addTwo = hyper({
  description: "Add two numbers",
  args: {
    a: z.number().describe("First number"),
    b: z.number().describe("Second number"),
  },
  handler: async ({ a, b }) => {
    return a + b
  },
})
```

This `addTwo` object now has a bunch of metadata associated with it.

#### Defining a group of functions

```typescript
export const functions = hyperStore({
  parse_date: hyper({
    description: "Parse a date from an ISO string",
    args: {
      date: z
        .string()
        .describe(
          "Date to parse, in ISO 8601 calendar date extended format",
        ),
    },
    handler({ date }) {
      dateObj = parseISO(date);
    },
  }),
  add_hours: hyper({
    description: "Add hours to date",
    args: {
      hours: z.number().describe("Hours to add"),
    },
    handler({ hours }) {
      dateObj = addHours(dateObj, hours);
    },
  }),
  add_days: hyper({
    description: "Add days to date",
    args: {
      days: z.number().describe("Days to add"),
    },
    handler({ days }) {
      dateObj = addDays(dateObj, days);
    },
  }),
  get_date: hyper({
    description: "Format processed date in localized format",
    args: {
      timezone: z
        .string()
        .describe("Timezone to convert date to before formatting."),
    },
    handler() {
      return intlFormat(dateObj, {
        dateStyle: "full",
        timeStyle: "full",
        timeZone: "Asia/Singapore",
      });
    },
  }),
});
```

#### Modifying existing function group

Imperatively define a new function on the existing group:

```typescript
functions.set("get_place_details", hyper({
  description: "Get details of place using the Google Maps API.",
  args: {
    place_id: z
      .string()
      .describe(
        "A textual identifier that uniquely identifies a place, returned from a Place Search.",
      ),
  },
  async handler({ place_id }) {
    const client = new GoogleMapsClient();
    const response = await client.placeDetails({
      params: {
        place_id,
        key: Env.GOOGLE_MAPS_API_KEY,
      },
    });

    return response.data;
  },
}));
```

#### Converting HyperFunctions to Tools

You can convert HyperFunctions to OpenAI Chat Completion Tools using the `asTools` method. This is useful for integrating HyperFunctions with OpenAI's tools.

```typescript
const completion = await openai.chat.completions.create({
  max_tokens: 4096,
  messages: [...history.get(), ...current.get()],
  model,
  tool_choice: "auto",
  tools: functions.asTools(),
});
```

#### Running HyperFunctions

You can run functions after getting LLM outputs as such:

```typescript
if (chosen.message.tool_calls) {
  for (const toolCall of chosen.message.tool_calls) {
    const response = await functions.callTool(toolCall, context);
    console.log(response) // output of function call (type: unknown)
  }
}
```

### Contributing

We welcome contributions to the HyperFunction library to improve its functionality, fix bugs, and add new features. Before contributing, please take a moment to review the guidelines below to ensure a smooth and collaborative development process.

#### Reporting Issues

If you encounter any bugs, issues, or have feature requests, please open an issue on the GitHub repository. When reporting an issue, please provide as much detail as possible, including:

- Description of the issue
- Steps to reproduce the issue
- Expected behavior
- Any relevant code snippets or error messages
- Environment details (operating system, Node.js version, etc.)

#### Code Contribution

##### Fork and Clone

1. Fork the HyperFunction repository to your GitHub account.
2. Clone the forked repository to your local machine.

```shell
git clone https://github.com/your-username/hyperfunction.git
```

3. Navigate to the cloned repository.

```shell
cd hyperfunction
```

##### Branching

Create a new branch for your contributions.

```shell
git checkout -b <username>/<feature-branch>
```

### Conclusion

The HyperFunction library simplifies the creation, storage, and usage of hyperfunctions in your projects. Explore the provided functions and methods to harness the full power of HyperFunctions in your applications.
