integration
===========

[![ISC License](http://img.shields.io/badge/license-ISC-blue.svg)](http://copyfree.org)

This contains integration tests which make use of the
[rpctest](rpctest/README.md)
package to programmatically drive nodes via RPC.

The tests are **not** normally run when targeting all soterd tests (`go test -v ./...`), because they use a build tags that `go test` doesn't match by default.

## Running integration tests

To target the integration tests, specify the `rpctest` build tag when calling `go test`:
```shell
go test -v -tags rpctest github.com/soteria-dag/soterd/integration
```

If there's an integration test that takes a particularly long amount of time to complete (depending on the system running the test), you can ask `go test` to allow it more time to complete than the current default of 10 minutes:
```shell
go test -v -timeout 1h -tags rpctest github.com/soteria-dag/soterd/integration
```

## Other integration test build tags

Some integration tests define build tags in addition to `rpctest`, to help you target integration tests with something in common. These additional build tags are:

|build tag|purpose|
|----|----|
|`dag`|Run dag-related integration tests|
|`daggen`|Run block-generation integration tests|
|`dagsync`|Run synchronization integration tests|
|`dagtxn`|Run block-transaction integration tests|

## Keeping integration test logs

Integration tests call `rpctest.New()` to create soterd nodes. The `keepLogs` parameter for that method determines if the logs generated by the soterd node will be kept after the node has been shut down.

The `TestDAGGen` integration test (in `gen_dag_test.go`) demonstrates the use of this parameter. It also calls `(*rpctest.Harness).LogDir()` to get the soterd node's log directory, so that they can be displayed in test output.

### Log file rotation

Soterd rotates log files it generates. Log file rotation settings are defined in `log.go`. The current settings are:

|setting|value|
|----|----|
|maximum log file size before rotation|10 MB|
|maximum rolls|3|

With these settings soterd will keep up to the last 30MB of logs spread across 3 files.

While debugging integration tests you might be passing `--debuglevel=debug` argument to `rpctest.New()` calls to increase logging verbosity. This may result in lost log data due to rotation. If this happens you could temporarily increase the rotation limits in `log.go` like this:
```diff
diff --git a/log.go b/log.go
index cc6b207f..147faf8b 100644
--- a/log.go
+++ b/log.go
@@ -120,7 +120,7 @@ func initLogRotator(logFile string) {
                fmt.Fprintf(os.Stderr, "failed to create log directory: %v\n", err)
                os.Exit(1)
        }
-       r, err := rotator.New(logFile, 10*1024, false, 3)
+       r, err := rotator.New(logFile, 3000*1024, false, 3)
        if err != nil {
                fmt.Fprintf(os.Stderr, "failed to create file rotator: %v\n", err)
                os.Exit(1)
```

## Integration test incompatibilities with block dag

Some integration tests currently fail when running against block dag implementation (tests in `bip0009_test.go`). BIP features may be reviewed as development continues, and updated or disabled based on what makes sense to keep.


## License

This code is licensed under the [copyfree](http://copyfree.org) ISC License.