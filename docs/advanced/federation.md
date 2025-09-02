---
title: GraphQL Federation
---

# GraphQL Federation Support

This package supports **Apollo GraphQL Federation** for building distributed GraphQL architectures. When federation is enabled, the generator automatically adds `@key` directives to your entity types based on the ID fields in your Prisma models.

## How to enable Federation

Add the `useFederation` option to your Prisma generator configuration:

```prisma {3}
generator typegraphql {
  provider      = "typegraphql-prisma"
  useFederation = true
}
```

## What it does

When federation is enabled, the generator automatically:

- **Generates `@key` directives** for entity definitions based on your Prisma model ID fields
- **Creates federation-compatible TypeGraphQL types** that work with Apollo Federation
- **Uses all ID fields** as federation keys (supports both single and composite keys)

## Example output

### Single ID field

For a Prisma model with a single ID field:

```prisma
model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  name  String
}
```

The generator will create:

```typescript
@TypeGraphQL.Directive(`@key(fields: "id")`)
@TypeGraphQL.ObjectType("User")
export class User {
  @TypeGraphQL.Field(_type => TypeGraphQL.Int, {
    nullable: false
  })
  id!: number;

  @TypeGraphQL.Field(_type => String, {
    nullable: false
  })
  email!: string;

  @TypeGraphQL.Field(_type => String, {
    nullable: false
  })
  name!: string;
}
```

### Composite ID fields

For models with composite ID fields:

```prisma
model UserRole {
  userId Int
  roleId Int
  user   User @relation(fields: [userId], references: [id])
  role   Role @relation(fields: [roleId], references: [id])

  @@id([userId, roleId])
}
```

The generator will create:

```typescript
@TypeGraphQL.Directive(`@key(fields: "userId,roleId")`)
@TypeGraphQL.ObjectType("UserRole")
export class UserRole {
  // ... generated fields
}
```

## Federation Schema

This enables your types to be used as entities in your federated GraphQL schema. The generated `@key` directives tell the Apollo Federation gateway which fields can be used to uniquely identify and resolve entities across different subgraphs.

## Usage with Apollo Federation

Once generated, you can use these types in your Apollo Federation subgraph:

```typescript
import { buildSubgraphSchema } from '@apollo/subgraph';
import { buildSchema } from 'type-graphql';
import { User } from '@generated/type-graphql';

const schema = buildSubgraphSchema(
  buildSchema({
    resolvers: [
      // Your resolvers that include User
    ],
    orphanedTypes: [User], // Include federated entities
  })
);
```

## Notes

- The `@key` directive is only added when `useFederation` is explicitly set to `true`
- All fields marked with `@id` in your Prisma schema will be included in the federation key
- The feature works with both simple and composite primary keys
- Generated types are fully compatible with Apollo Federation specifications
