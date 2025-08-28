---
title: "Error handling in drizzle ORM"
sidebar_position: 3
---

# Notes on error handling in drizzle ORM


### Background:

Drizzle depends on database drivers to perform actual operations on the database. For `postgrsql` database ,drizzle supports [postgresjs](https://github.com/postgres/postgres) and [node-postgres (also called pg)](https://www.npmjs.com/package/pg) as [documented on their website](https://orm.drizzle.team/docs/get-started-postgresql).


Now the problem is, since drizzle does not perform the database operations itself it gets success/failure information about the operation through **errors/messages** returned by the underlying driver.

The drizzle's session layer is driver agnostic. So the database interation code [looks like this](https://github.com/drizzle-team/drizzle-orm/blob/37d059f95ebe4ca7da6e60415de0164c729a8454/drizzle-orm/src/pg-core/session.ts#L71C4-L71C25):

```typescript
try {
    return await query();
}
catch (e) {
    throw new DrizzleQueryError(queryString, params, e as Error);
}
```

Basically, it just tries to run the query using the database driver provided, and in case, any error is thrown at a lower level, it just:

- throws a new [`DrizzleQueryError`](https://github.com/drizzle-team/drizzle-orm/blob/37d059f95ebe4ca7da6e60415de0164c729a8454/drizzle-orm/src/errors.ts#L3)
- the original error is attached to the [`cause`](https://github.com/drizzle-team/drizzle-orm/blob/37d059f95ebe4ca7da6e60415de0164c729a8454/drizzle-orm/src/errors.ts#L9) field

There are several problems with this approach:

- the type of error thrown cannot be ascertained
- drizzle is providing no abstraction, and the developer needs to figure out **what went wrong** by understanding the error thrown by the underlying driver.

Though I think this is a difficult problem, because:

- if drizzle depends on individual database drivers and throws errors based on them, then it is no longer **driver agnostic**.
- it is difficult to track down each possible error the `database` driver and throw. And since, it must support multiple drivers, it needs to define a common interface, that can cover all possible errors.

I think the best approach out here, is to define a common interface for all Errors (can expand DrizzleQueryError to include more fields). And then, in the catch block above, try to find out what went wrong and populate as must information we have before we throwing the `DrizzleQueryError`.

### Temprorary solution

I'm using [`node-postgres`](https://www.npmjs.com/package/pg-protocol).  And I found out that it actually uses `pg-protocol` under the hood.

Upon failure, it throws a [`DatabaseError`](https://github.com/brianc/node-postgres/blob/917478397b0cfbb95f0275e1974a72bb581b07a9/packages/pg-protocol/src/messages.ts#L97) with the error code and message in accordance with [postgres error codes as described here](https://www.postgresql.org/docs/current/errcodes-appendix.html).

So I build the following solution, to detect errors:


```typescript
import { DrizzleQueryError } from "drizzle-orm";
import { DatabaseError }     from "pg-protocol";

// Relevant codes from: https://www.postgresql.org/docs/current/errcodes-appendix.html

export enum OPSTATUS {
    SUCCESS,
    // integrity violations
    FOREIGN_KEY_VIOLATION=23503,
    UNIQUE_VIOLATION=23505,
    CHECK_VIOLATION=23514,
    NOT_NULL_VIOLATION=23502,

    // transaction failure
    INVALID_TRANSACTION_STATE=25000,

    // connection failure
    CONNECTION_DOES_NOT_EXIST=8006,
    CONNECTION_FAILURE=8006,

    // other
    UNKNOWN_FAILURE=-1,
}

// ...

    try {
        const insertedUser = await db.insert(usersSchema).values(user).returning();

        if ( insertedUser[0] === undefined ) {
            throw new Error("Unknown Failure"); // handled below
        }

        return insertedUser[0];
    }

    catch (err: any) {

        // if error has not originated from pg driver, no useful information can be extracted
        if ( !(err instanceof DrizzleQueryError) || !(err.cause instanceof DatabaseError) ) {
            return {
                success: false,
                status: OPSTATUS.UNKNOWN_FAILURE,
                message: "Unknown Failure"
            }
        }

        // ...

        // try to find out what went wrong?
        switch (errCode) {

            case OPSTATUS.UNIQUE_VIOLATION: {
                recommendedHttpResponseCode = StatusCodes.CONFLICT;
                message = "User already exists";
                break;
            }
            case OPSTATUS.CONNECTION_FAILURE: {
                recommendedHttpResponseCode = StatusCodes.SERVICE_UNAVAILABLE;
                message = "Broken connection to database server";
                break;
            }
            // and so on..
        }

        // return approapriate response
    }

```
