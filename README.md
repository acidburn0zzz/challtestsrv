# Challenge Test Server

The `challtestsrv` package offers a library/command that can be used by test
code to respond to HTTP-01, DNS-01, and TLS-ALPN-01 ACME challenges. The
`challtestsrv` package can also be used as a mock DNS server letting
developers mock `A`, `AAAA`, and `CAA` DNS data for specific hostnames.

**Important note: The `challtestsrv` command and library are for TEST USAGE
ONLY. It is trivially insecure, offering no authentication. Only use
`challtestsrv` in a controlled test environment.**

For example this package is used by the Boulder
[`load-generator`](https://github.com/letsencrypt/boulder/tree/9e39680e3f78c410e2d780a7badfe200a31698eb/test/load-generator)
command to manage its own in-process HTTP-01 challenge server.

### Usage

Create a challenge server responding to HTTP-01 challenges on ":8888" and
DNS-01 challenges on ":9999" and "10.0.0.1:9998":

```
  import "github.com/letsencrypt/pebble/challtestsrv"

  challSrv, err := challtestsrv.New(challsrv.Config{
    HTTPOneAddr: []string{":8888"},
    DNSOneAddr: []string{":9999", "10.0.0.1:9998"},
  })
  if err != nil {
    panic(err)
  }
```

Run the Challenge server and subservers:
```
  // Start the Challenge server in its own Go routine
  go challSrv.Run()
```

Add an HTTP-01 response for the token `"aaa"` and the value `"bbb"`, defer
cleaning it up again:
```
  challSrv.AddHTTPOneChallenge("aaa", "bbb")
  defer challSrv.DeleteHTTPOneChallenge("aaa")
```

Add a DNS-01 TXT response for the host `"_acme-challenge.example.com."` and the
value `"bbb"`, defer cleaning it up again:
```
  challSrv.AddDNSOneChallenge("_acme-challenge.example.com.", "bbb")
  defer challSrv.DeleteHTTPOneChallenge("_acme-challenge.example.com.")
```

Get the history of requests processed by the challenge server:
```
requestHistory := challSrv.RequestHistory()
```

Clear the history of requests:
```
challSrv.ClearRequestHistory()
```

Stop the Challenge server and subservers:
```
  // Shutdown the Challenge server
  challSrv.Shutdown()
```

For more information on the package API see Godocs and the associated package
sourcecode.
