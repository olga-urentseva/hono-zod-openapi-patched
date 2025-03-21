# hono-zod-openapi-patched

A patch for [@hono/zod-openapi](https://github.com/honojs/middleware/tree/main/packages/zod-openapi) that adds support for custom media types in request bodies, following the HTTP spec 8.3.

> [!WARNING]
> This patched library was primarily developed for a personal project to enable the use of any and multiple media types in HTTP request bodies. I'd love to see this behavior incorporated into the original `@hono` or `@hono/zod-openapi` packages.
> I'd prefer this to be a private package, because of many things I don't like here, but I decided to publish it because someone might need this like I do.

## Problem

The original `@hono` and `@hono/zod-openapi` packages support JSON and Form content types for request bodies. This limitation prevents you from using custom media types or other standard media types that may be needed for your API.

Another issue is that the `@hono` libraries limit the number of media types per route to just one. This makes it impossible to configure a route to work with different content types (e.g., JSON, images, etc.). Each route is restricted to one content type. Despite the fact that you can pass configuration with multiple content types, `Hono` will ignore all except the first one (and still it suppose to be JSON or Form).

Note that these limitations are implemented in `@hono` itself, so extended libraries like `@hono/zod-openapi` simply inherit these limitations.

## Solution

This patched library extends the original implementation of `@hono` and `@hono/zod-openapi`, but uses a different approach to validate request bodies. It supports any media type and any quantity of media types in request bodies following HTTP specification. It properly validates the incoming content based on the specified media type and schema, and makes both the media type and validated data available in your handlers.

## Installation

```bash
# Using npm
npm install hono-zod-openapi-patched

# Using pnpm
pnpm add hono-zod-openapi-patched

# Using yarn
yarn add hono-zod-openapi-patched

```

## Usage

Use this package as a drop-in replacement for @hono/zod-openapi:

```ts
import { OpenAPIHono, createRoute } from "hono-zod-openapi-patched";
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
