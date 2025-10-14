# Global Cache example with Playwright

An example of using [@global-cache/playwright](https://github.com/vitalets/global-cache) for cached authentication in the Playwright tests.

This approach is more efficient than the [dependency project](https://playwright.dev/docs/auth#basic-shared-account-in-all-tests), because authentication is performed on-demand and doesn't require extra project in your config.

## Details
- `test/auth.spec.ts` - tests for authenticated page
- `test/no-auth.spec.ts` - tests for non-authenticated page
- `test/helpers/fixtures.ts` - overwrites `storageState` fixture to lazily perform authetication
- `test/helpers/auth.ts` - authentication helper

## Running all tests
When running all tests with 2 workers, authentication performed only in one worker (where it's really needed):
```
npx playwright test
```
Output:
```
Running 3 tests using 2 workers

  ✓  1 test/no-auth.spec.ts:5:5 › no-auth test @no-auth (2.0s)
  ✘  2 test/auth.spec.ts:4:5 › test 1 (4.6s)
Performing sing-in...
Worker 1, user is NOT authenticated.
Worker 0, user is authenticated.
  ✘  3 test/auth.spec.ts:11:5 › test 2 (1.8s)
Worker 2, user is authenticated.
```

## Running no-auth test 
When running only no-auth test, authentication is skipped and test runs faster:
```
npx playwright test no-auth.spec.ts
```
Output:
```
Running 1 test using 1 worker

  ✓  1 test/no-auth.spec.ts:5:5 › no-auth test @no-auth (2.1s)
Worker 0, user is NOT authenticated.
```

## Running on shards
When running on 2 shards, authentication is performed only on one shard:

Shard 1:
```
npx playwright test --shard=1/2
```
Output:
```
Running 2 tests using 1 worker, shard 1 of 2

  ✘  1 test/auth.spec.ts:4:5 › test 1 (4.3s)
Performing sing-in...
Worker 0, user is authenticated.
  ✘  2 test/auth.spec.ts:11:5 › test 2 (2.0s)
Worker 1, user is authenticated.
```

Shard 2:
```
npx playwright test --shard=2/2
```
Output:
```
Running 1 test using 1 worker, shard 2 of 2

  ✓  1 test/no-auth.spec.ts:5:5 › no-auth test @no-auth (2.3s)
Worker 0, user is NOT authenticated.
```