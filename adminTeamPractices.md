# Lessons learned and recommended practices

## Central type file

## Types and jsdocs integration
```typescript
export interface SearchParameters {
  limit: number;
  /** The status of the users we wish to fetch */
  status?: Status;
  /** The column to sort by */
  sortBy: string;
  sortOrder: 'ASC' | 'DESC';
  /** Email of last user on the previous page. Used for paging */
  seekEmail?: string;
  /** query string for partial text search by user name and email */
  q?: string | null;
  /** Site filter */
  domainCname?: string | null;
  roleId?: number | null;
  /** Count the total number of users. Not limited by limit.  */
  includeCount?: boolean;
}
```
This allows your IDE to pull the types with their definitions to be references in other parts of the code. Giving further explanation of the uses.

## actions and return types

Recommend to use `typesafe-actions` and generate return types for all actions.

```typescript
export type organizationActionType =
  | fetchOrgAction
  | fetchOrgSuccessAction
  | fetchOrgFailedAction
  | initialLoadAction;
```

which can then be passed to the reducer.

## selectors and return types

Selectors like ordinary functions should have explicit return types.

```typescript
export const getOrg = (state: ApplicationState): Organization | {} => get(state, 'organization', {});

export const getUsers = (state: ApplicationState): Users => get(state, 'users', defaultUserState);

export const getSites = (state: ApplicationState): Domain[] => get(getOrg(state), 'domains', []);

export const getRolesList = (state: ApplicationState): Role[] => get(getOrg(state), 'userRoles', []);
```

## Getting resources and setting return type

```typescript
    // fetch will return an `any` type that you can cast to the type you expect
    const payload: {data: Data, error: Error} = await fetchAndParseStream('https://www.website.com/some/api'); 

    // or....

    interface MyInterface {
        whatever: string;
    }
    
    const result = await fetchAndParseStream('https://www.website.com/some/api') as MyInterface;
```

We can also look into using libraries to do this: https://github.com/gcanti/io-ts seems to be a popular one

## Explicit return types on selectors and functions (things beside sagas, components, etc..)
 
 Explicitly setting the return type of ordinary functions and specifically selectors is useful to test your assumptions.

## Trade-offs with PropTypes

`PropTypes` pros:
- Runtime validation that components are getting what they expect can make it easy to find missing props
    - Note: There are ways to do this in TS as well; https://github.com/gcanti/io-ts and other libraries

`PropTypes` cons:
- No real validation that components are being used how they're expected to at compile-time
- No errors or warnings until you run the code _and_ render the component; this means that changes to props can go un-checked for sometimes many releases until somebody hits an infrequently-used component
- No IDE integration at all; no autocomplete or docs or anything
- Not enforced in any way; you can totally just not include `propTypes` for a component (depending on where you're at in our frontend, ESLint may or may not enforce that all `props` are defined)

TS pros:
- Compile-time assurance that you're passing all required props and that they're the correct types
- Easier to refactor props that a component uses; adding a new required prop? The compiler will tell you _everywhere_ that you have to add it in (vs. with `propTypes` having to grep for usages of a component and hope you get them all)

## When to rewrite JS to TS?

## How to rewrite

Work done for the TS lunch-n-learn has a great example of rewriting JS to TS:
https://github.com/socrata/typescript-demo/tree/master/components

(see `ReduxExampleJS` vs `ReduxExampleTS`)
 
## Avoid `any`

## Avoiding "undefined is not a function" errors

```typescript
interface SomeInterface {
    something?: string;
}

// `something` is optional so this is fine
const whatever: SomeInterface = { }

// COMPILE ERROR: `something` can be undefined
whatever.something.length();

// Fixed! Enforced that we have `something`
if (whatever.something) {
    whatever.something.length();
}
```
 
  *   What information would you like your colleagues to know before voting on TypeScript? 
   - Reduces tribal knowledge
   - Simplified refactoring
   - More robust tests
   - Good IDE support and assistance with common edge case handling.
   - Think of TypeScript as a super fancy configurable linter
   - Thinking of types beforehand can lead to better overall code (thinking before writing)
   - Porting is fairly simple and straight forward

     *   What was your experience like porting Org Dash to TypeScript? 
     *   Tooling issues? N/A (difficult to integrate into our complex build but the work is done)
     TS-Linting isn't as robust
     *   Best practices? See above.
  *   What information do *you* still need to be comfortable making a decision? None. Use it.
  *   What degree of adoption would *you* like to see? 100% of future work.
