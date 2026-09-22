# curl cheatsheet

curl 8.x. `-s` silent, `-S` show errors even when silent, `-v` verbose.

## Basic requests

```bash
curl https://api.example.com/users
curl -o out.json https://api.example.com/users   # save to file
curl -O https://example.com/file.tar.gz          # save with remote name
curl -L https://example.com                      # follow redirects
curl -I https://example.com                      # HEAD, headers only
curl -s -o /dev/null -w '%{http_code}\n' https://example.com
```

## Methods / bodies

```bash
curl -X POST https://api/x
curl -X PUT https://api/x/1
curl -X DELETE https://api/x/1
curl -X PATCH https://api/x/1

curl -d 'a=1&b=2' https://api/x                       # form-encoded (POST)
curl -d '{"a":1}' -H 'Content-Type: application/json' https://api/x
curl --json '{"a":1}' https://api/x                   # curl 7.82+ shorthand
curl -d @body.json -H 'Content-Type: application/json' https://api/x
curl -F 'file=@photo.png' https://api/upload          # multipart file upload
curl -F 'name=ann' -F 'file=@a.pdf' https://api/upload
```

`-d` implies POST and sets `Content-Type: application/x-www-form-urlencoded`.

## Headers / auth

```bash
curl -H 'Authorization: Bearer <token>' https://api/x
curl -u user:pass https://api/x                       # basic auth
curl -u user https://api/x                            # prompts for password
curl --oauth2-bearer "$TOKEN" https://api/x
curl -H 'Accept: application/json' -H 'X-Api-Version: 2' https://api/x
curl -b 'session=abc' https://api/x                   # send cookie
curl -c cookies.txt -b cookies.txt https://api/x      # save + reuse jar
```

## Debugging

```bash
curl -v https://api/x                    # request + response headers
curl -vvv https://api/x                  # even more
curl -i https://api/x                    # include response headers in output
curl --trace-ascii trace.txt https://api/x
curl -s -D - -o /dev/null https://api/x  # headers only, body discarded
curl -w '\n%{http_code} %{time_total}s\n' -o /dev/null -s https://api/x
curl --resolve api.example.com:443:10.0.0.5 https://api.example.com  # fake DNS
curl -k https://self-signed.local        # skip cert check (insecure, testing only)
curl --cacert ca.pem https://internal    # trust a private CA
```

Timing breakdown:

```bash
curl -o /dev/null -s -w \
  'dns:%{time_namelookup} conn:%{time_connect} tls:%{time_appconnect} ttfb:%{time_starttransfer} total:%{time_total}\n' \
  https://example.com
```

## Common options

```bash
--connect-timeout 5          # connect phase only
--max-time 30                # whole operation
--retry 3 --retry-delay 1    # retry transient failures
--retry-all-errors           # retry even on 4xx/5xx (curl 7.71+)
-x http://proxy:8080         # use a proxy
--proxy-user user:pass
-sS                          # silent but show errors
-f                           # fail (exit nonzero) on HTTP >= 400
--compressed                 # send Accept-Encoding and decompress
-#                           # progress bar
```

`-f` is what makes curl usable in scripts: without it, curl exits 0 even on a
500, and you get the error page as "success".

## Uploading / downloading

```bash
curl -T file.txt ftp://host/           # upload
curl -T big.iso https://upload/x       # PUT
curl -C - -O https://host/big.iso      # resume a partial download
curl -r 0-1023 https://host/file       # byte range
curl --limit-rate 1M -O https://host/big.iso
```

## Scripting patterns

```bash
# fail the script on HTTP errors, capture body
body=$(curl -fsSL https://api/x) || { echo "request failed" >&2; exit 1; }

# check status code
code=$(curl -s -o /dev/null -w '%{http_code}' https://api/x)
[[ "$code" == "200" ]] || echo "got $code"

# post and pipe to jq
curl -s -H 'Content-Type: application/json' -d @in.json https://api/x | jq .
```

## Notes

- `-d @file` reads the body from a file; `-d @-` reads from stdin.
- `--json` sets both `Content-Type: application/json` and `Accept: application/json`.
- `-k` disables cert verification. Fine for a quick test, never in prod code.
- Exit codes matter: 6 = couldn't resolve host, 7 = connection refused,
  28 = timeout. Useful in scripts.
