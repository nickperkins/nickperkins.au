---
title: "Building Interactive MCP Tools with Elicitations: A Practical Guide"
date: 2025-07-11T22:00:00+10:00
draft: false

categories: [Software Development]
tags: [MCP, Model Context Protocol, TypeScript, AI tools, interactive tools, elicitation]
toc: false
author: "Nick Perkins"
---
The Model Context Protocol (MCP) is revolutionising how we build AI tools, but a  powerful new features recently added to the protocol will be a game changer: elicitations. These enable your MCP tools to have interactive conversations with users, gathering input dynamically rather than requiring all parameters upfront. Let me show you how to build engaging, interactive MCP tools using a practical ice cream topping recommender as our example.

## What Are MCP Elicitations?

Elicitations in MCP allow tools to prompt users for additional information when needed. Instead of failing when required parameters are missing, your tool can ask the user to provide them through a structured interface. This creates a more natural, conversational experience with AI assistants.

Think of it like this: rather than requiring users to specify every parameter in their initial request, your tool can guide them through the process step by step, asking clarifying questions along the way.

## The Ice Cream MCP Server: An Example Implementation

I built a minimal MCP server that demonstrates elicitations perfectly. The `ice_cream_topping_recommender` tool suggests toppings based on ice cream flavours, but here's the clever part: if no flavour is provided, it doesn't fail—it asks the user to choose one.

{{< github repo="nickperkins/ice-cream-mcp" >}}

## Architecture Overview

The server follows a clean, modular architecture:

```typescript
export class IceCreamMcpServer {
  private server: Server;
  private tool: IceCreamToppingRecommenderTool;

  // Registers tool and sets up MCP protocol handlers
  // Provides elicitation function to tools
  // Handles errors and server lifecycle
}
```

Each tool implements a consistent interface:

```typescript
class IceCreamToppingRecommenderTool {
  getName(): string
  getDescription(): string
  getInputSchema(): ZodSchema
  execute(args, elicitInput?): Promise<MCPResponse>
}
```

The magic happens in the `execute` method, where the optional `elicitInput` function enables interactive behaviour.

## How Elicitations Work in Practice

Here's the core elicitation logic:

```typescript
if (!flavour && elicitInput) {
  const jsonSchema = {
    type: "object",
    properties: {
      flavour: {
        type: "string",
        enum: ["vanilla", "strawberry", "chocolate"],
        description: "Choose a flavour: vanilla, strawberry, or chocolate."
      }
    },
    required: ["flavour"]
  };

  const result = await elicitInput({
    message: "Which ice cream flavour do you want toppings for?",
    requestedSchema: jsonSchema,
  });

  if (result.cancelled) {
    return { content: [{ type: "text", text: "❌ Operation cancelled." }] };
  }

  flavour = result.data.flavour;
}
```

When a user invokes the tool without specifying a flavour, the tool:

1. **Detects missing required input** (`!flavour`)
2. **Checks if elicitation is available** (`elicitInput`)
3. **Presents a schema-driven interface** with enum options
4. **Waits for user selection** before proceeding
5. **Handles cancellation gracefully** if the user backs out

## The User Experience

From the user's perspective, the interaction flows naturally:

**User**: "I want ice cream topping suggestions"

**Tool**: "Which ice cream flavour do you want toppings for? (vanilla, strawberry, or chocolate)"

**User selects**: "vanilla"

**Tool**: "🍦 For vanilla ice cream, the best toppings are: caramel sauce, rainbow sprinkles, crushed biscuits."

Here's what this looks like in practice. First, the tool prompts for the missing flavour parameter:

{{< imgproc img="demo-1.png" command="Resize" options="800x" alt="Screenshot showing the MCP tool prompting the user to select an ice cream flavour with a dropdown interface containing vanilla, strawberry, and chocolate options" >}}The elicitation interface prompting for flavour selection{{< /imgproc >}}

After the user selects their flavour, the tool provides personalised recommendations:

{{< imgproc img="demo-2.png" command="Resize" options="800x" alt="Screenshot showing the MCP tool's response with ice cream topping recommendations for the selected flavour, displaying the final output with emoji and formatted text" >}}The tool providing tailored topping recommendations{{< /imgproc >}}

This conversational flow feels much more natural than requiring the user to format their request perfectly from the start.

## Technical Implementation Details

### Type Safety with Zod

The server uses Zod for robust input validation:

```typescript
const IceCreamFlavourSchema = z.object({
  flavour: z.enum(["vanilla", "strawberry", "chocolate"]).optional(),
});
```

This ensures type safety while allowing optional parameters that can be gathered through elicitation.

### Error Handling

The implementation includes comprehensive error handling:

* Global error handlers for unhandled rejections
* Tool-specific error formatting with emoji prefixes
* Graceful degradation when elicitation is cancelled
* Proper error propagation through the MCP protocol

### Integration Points

The server works seamlessly with both Claude Desktop and VS Code through standard MCP integration:

```json
{
  "servers": {
    "ice-cream-topping-recommender": {
      "type": "stdio",
      "command": "node",
      "args": ["${workspaceFolder}/build/index.js"]
    }
  }
}
```

## Why This Pattern Matters

### Better User Experience

Elicitations transform rigid, parameter-heavy tool calls into natural conversations. Users don't need to remember exact parameter names or formats—the tool guides them through the process.

### Reduced Cognitive Load

Instead of requiring users to understand your tool's complete API upfront, they can discover capabilities organically through interaction.

### Error Prevention

By validating input through structured schemas, elicitations prevent many common user errors before they occur.

## Building Your Own Interactive MCP Tools

Ready to add elicitations to your MCP tools? Here's how:

### 1. Design Your Schema

Define what information your tool needs and what's optional:

```typescript
const MyToolSchema = z.object({
  requiredParam: z.string(),
  optionalParam: z.string().optional(),
});
```

### 2. Implement Elicitation Logic

Check for missing parameters and prompt for them:

```typescript
if (!optionalParam && elicitInput) {
  const result = await elicitInput({
    message: "Please provide the missing parameter:",
    requestedSchema: {
      type: "object",
      properties: {
        optionalParam: {
          type: "string",
          description: "Description of what this parameter does"
        }
      },
      required: ["optionalParam"]
    },
  });

  if (!result.cancelled) {
    optionalParam = result.data.optionalParam;
  }
}
```

### 3. Handle All Scenarios

Always account for:
* Happy path with all parameters provided
* Elicitation flow for missing parameters
* User cancellation
* Validation errors

## Real-World Applications

While ice cream toppings make for a fun demo, elicitations have serious applications:

* **Configuration wizards** that guide users through complex setups
* **Data collection tools** that gather information step by step
* **Diagnostic tools** that ask follow-up questions based on initial symptoms
* **Content generators** that refine requirements through conversation

## The Future of Interactive AI Tools

MCP elicitations represent a shift toward more conversational, user-friendly AI tools. Instead of requiring users to learn complex APIs, tools can guide users through natural interactions, making AI assistance more accessible and effective.

The ice cream MCP server might be simple, but it demonstrates a powerful pattern that can transform how we build AI tools. By embracing elicitations, we can create tools that are not just functional, but genuinely enjoyable to use.

Start building your own interactive MCP tools today—your users will thank you for the improved experience, and you'll discover new possibilities for AI-human collaboration.
