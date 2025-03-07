# hono-zod-openapi-patch

A patch for [@hono/zod-openapi](https://github.com/honojs/middleware/tree/main/packages/zod-openapi) that adds support for custom media types in request bodies, following the HTTP spec 8.3.

## Problem

The original `@hono/zod-openapi` package only supports JSON and Form content types for request bodies. This limitation prevents you from using custom media types or other standard media types that may be needed for your API.

## Solution

This patch extends the original implementation to support any media type in request bodies. It properly validates the incoming content based on the specified media type and schema, and makes both the media type and validated data available in your handlers.

## Installation

```bash
# Using npm
npm install hono-zod-openapi-patch

# Using pnpm
pnpm add hono-zod-openapi-patch

# Using yarn
yarn add hono-zod-openapi-patch

```

## Usage

Use this package as a drop-in replacement for @hono/zod-openapi:

```ts
import { OpenAPIHono, createRoute } from "hono-zod-openapi-patch";
import { z } from "zod";

const app = new OpenAPIHono();

// Define a route that accepts media types
const route = createRoute({
  method: "post",
  path: "/data",
  request: {
    body: {
      content: {
        "application/json": {
          schema: z.object({
            name: z.string(),
          }),
        },
        "application/xml": {
          schema: z.string(),
        },
        "text/csv": {
          schema: z.string(),
        },
        "application/x-my-custom-type": {
          schema: z.number(),
        },
      },
    },
  },
  responses: {
    200: {
      content: {
        "application/json": {
          schema: z.object({
            success: z.boolean(),
            mediaType: z.string(),
          }),
        },
      },
      description: "Success",
    },
  },
});

app.openapi(route, (c) => {
  // Get the validated body and its media type
  const validatedBody = c.get("validatedBody");
  const { mediaType, data } = validatedBody;

  // Now you can process the data based on the media type
  console.log(`Received data with media type: ${mediaType}`);

  return c.json(
    {
      success: true,
      mediaType: mediaType,
    },
    200
  );
});
```

## NOTE!

This project includes an update to how body validation works. Other behaviors remain unchanged, so you can still use constructs like:

```ts
const { id } = c.req.valid("param");
```

BUT never

```ts
const { title } = c.req.valid("json");
```

For more details, refer to the hono zod openapi documentation.

## P.S.

This patch was primarily developed for a personal project to enable the use of any media type in HTTP requests. I’d love to see this behavior incorporated into the original package.
I'd prefer this to be a private package, because of many things I don't like here, but I decided to publish it because someone might need this.
