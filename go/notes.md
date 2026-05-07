# Container Security Fundamentals

```
$ git clone https://github.com/chainguard-dev/hello-melange-apko
Cloning into 'hello-melange-apko'...
remote: Enumerating objects: 455, done.
remote: Counting objects: 100% (46/46), done.
remote: Compressing objects: 100% (32/32), done.
remote: Total 455 (delta 25), reused 15 (delta 13), pack-reused 409 (from 1)
Receiving objects: 100% (455/455), 130.57 KiB | 2.25 MiB/s, done.
Resolving deltas: 100% (223/223), done.

$ cd hello-melange-apk/go
```

## 1. Containerization

### Create a single-stage Dockerfile

```
$ cat Dockerfile-single-unpatched
FROM golang:alpine
WORKDIR /app
COPY . .
RUN go build -o hello-server .
EXPOSE 8080
CMD ["./hello-server"]

$ docker build -t host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-unpatched -f Dockerfile-single-unpatched .
```


### Create a multi-stage Dockerfile

```
$ cat Dockerfile-multi-unpatched
FROM golang:alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o hello-server .

FROM scratch
WORKDIR /app
COPY --from=builder /app/hello-server ./hello-server
EXPOSE 8080
CMD ["./hello-server"]

$ docker build -t host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-unpatched -f Dockerfile-multi-unpatched .
```


### Use any base image of your choice

## 2. Security Analysis

### Scan both containers with a CVE scanner
### Document all vulnerabilities found
### Compare results between both approaches

```
$ grype host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-unpatched
 ✔ Vulnerability DB                [no update available]
 ✔ Loaded image                          host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-unpatched
 ✔ Parsed image                              sha256:6b56f658ddb4ceea03e5ef9a4d7161ae69c0da32e9b59315400706f485cdf41c
 ✔ Cataloged contents                               863259eb0b6f90af8b7b5f84ea8fee81e1d929fd92fe94767edd4e88e7dbc0f8
   ├── ✔ Packages                        [44 packages]
   ├── ✔ Executables                     [58 executables]
   ├── ✔ File metadata                   [240 locations]
   └── ✔ File digests                    [240 files]
 ✔ Scanned for vulnerabilities     [27 vulnerability matches]
   ├── by severity: 1 critical, 9 high, 17 medium, 0 low, 0 negligible
   └── by status:   24 fixed, 3 not-fixed, 0 ignored
NAME                        INSTALLED                           FIXED IN                           TYPE       VULNERABILITY        SEVERITY  EPSS           RISK
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.17.0                             go-module  GHSA-qppj-fm5r-hxr3  Medium    94.5% (99th)   58.3    KEV
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.23.0                             go-module  GHSA-4v7x-pqxf-cx7m  Medium    69.9% (98th)   36.0
golang.org/x/crypto         v0.0.0-20210711020723-a769d52b0f97  0.0.0-20231218163308-9d2ee975ef9f  go-module  GHSA-45x7-px36-x8w8  Medium    53.6% (98th)   29.2
golang.org/x/crypto         v0.0.0-20210711020723-a769d52b0f97  0.31.0                             go-module  GHSA-v778-237x-gjrc  Critical  30.3% (96th)   27.4
golang.org/x/crypto         v0.0.0-20210711020723-a769d52b0f97  0.35.0                             go-module  GHSA-hcg3-q754-cr77  High      0.6% (69th)    0.4
google.golang.org/protobuf  v1.28.0                             1.33.0                             go-module  GHSA-8r3f-844c-mc37  Medium    0.4% (60th)    0.2
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.7.0                              go-module  GHSA-vvpx-j8f3-3w6h  High      0.3% (49th)    0.2
github.com/gin-gonic/gin    v1.8.1                              1.9.1                              go-module  GHSA-2c4m-59x9-fr2g  Medium    0.4% (61st)    0.2
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.4.0                              go-module  GHSA-xrjj-mj9h-534m  Medium    0.3% (55th)    0.2
github.com/gin-gonic/gin    v1.8.1                              1.9.0                              go-module  GHSA-3vp4-m3rf-835h  Medium    0.2% (46th)    0.1
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.17.0                             go-module  GHSA-4374-p667-p6c8  High      0.2% (36th)    0.1
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.0.0-20210520170846-37e1c6afe023  go-module  GHSA-83g2-8m93-v3w7  High      0.1% (33rd)    0.1
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.0.0-20220906165146-f3363e06e74c  go-module  GHSA-69cg-p879-7622  High      0.1% (32nd)    < 0.1
golang.org/x/sys            v0.0.0-20210806184541-e5e7981a1069  0.0.0-20220412211240-33da011f77ad  go-module  GHSA-p782-xgp4-8hr8  Medium    0.2% (39th)    < 0.1
golang.org/x/crypto         v0.0.0-20210711020723-a769d52b0f97  0.0.0-20220314234659-1baeb1ce4c0b  go-module  GHSA-8c26-wmh5-6g9v  High      < 0.1% (25th)  < 0.1
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.38.0                             go-module  GHSA-vvgc-356p-c3xw  Medium    0.1% (30th)    < 0.1
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.13.0                             go-module  GHSA-2wrh-6pvc-2jm9  Medium    < 0.1% (26th)  < 0.1
golang.org/x/text           v0.3.6                              0.3.8                              go-module  GHSA-69ch-w2m2-3vjp  High      < 0.1% (16th)  < 0.1
golang.org/x/text           v0.3.6                              0.3.7                              go-module  GHSA-ppp9-7jff-5vj2  High      < 0.1% (16th)  < 0.1
busybox                     1.37.0-r30                                                             apk        CVE-2025-60876       Medium    < 0.1% (15th)  < 0.1
busybox-binsh               1.37.0-r30                                                             apk        CVE-2025-60876       Medium    < 0.1% (15th)  < 0.1
ssl_client                  1.37.0-r30                                                             apk        CVE-2025-60876       Medium    < 0.1% (15th)  < 0.1
golang.org/x/crypto         v0.0.0-20210711020723-a769d52b0f97  0.0.0-20211202192323-5770296d904e  go-module  GHSA-gwc9-m7rh-j2ww  High      < 0.1% (9th)   < 0.1
golang.org/x/crypto         v0.0.0-20210711020723-a769d52b0f97  0.45.0                             go-module  GHSA-j5w8-q4qc-rx2x  Medium    < 0.1% (13th)  < 0.1
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.0.0-20210428140749-89ef3d95e781  go-module  GHSA-h86h-8ppg-mxmh  Medium    < 0.1% (6th)   < 0.1
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.36.0                             go-module  GHSA-qxp5-gwg8-xv66  Medium    < 0.1% (6th)   < 0.1
golang.org/x/crypto         v0.0.0-20210711020723-a769d52b0f97  0.45.0                             go-module  GHSA-f6x5-jh6r-wrfv  Medium    < 0.1% (2nd)   < 0.1

$ grype host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-unpatched
 ✔ Loaded image                           host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-unpatched
 ✔ Parsed image                              sha256:ed3b39a5e53520a73dc8f5f850de449abe1ae0805eceb88b6411ef46194ebbc2
 ✔ Cataloged contents                               e77b47f63d7574c27545b23c455ae28bc0a63d6b04da7e4a69416aefd2f630d1
   ├── ✔ Packages                        [16 packages]
   ├── ✔ Executables                     [1 executables]
   ├── ✔ File metadata                   [1 locations]
   └── ✔ File digests                    [1 files]
 ✔ Scanned for vulnerabilities     [24 vulnerability matches]
   ├── by severity: 1 critical, 9 high, 14 medium, 0 low, 0 negligible
   └── by status:   24 fixed, 0 not-fixed, 0 ignored
NAME                        INSTALLED                           FIXED IN                           TYPE       VULNERABILITY        SEVERITY  EPSS           RISK
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.17.0                             go-module  GHSA-qppj-fm5r-hxr3  Medium    94.5% (99th)   58.3    KEV
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.23.0                             go-module  GHSA-4v7x-pqxf-cx7m  Medium    69.9% (98th)   36.0
golang.org/x/crypto         v0.0.0-20210711020723-a769d52b0f97  0.0.0-20231218163308-9d2ee975ef9f  go-module  GHSA-45x7-px36-x8w8  Medium    53.6% (98th)   29.2
golang.org/x/crypto         v0.0.0-20210711020723-a769d52b0f97  0.31.0                             go-module  GHSA-v778-237x-gjrc  Critical  30.3% (96th)   27.4
golang.org/x/crypto         v0.0.0-20210711020723-a769d52b0f97  0.35.0                             go-module  GHSA-hcg3-q754-cr77  High      0.6% (69th)    0.4
google.golang.org/protobuf  v1.28.0                             1.33.0                             go-module  GHSA-8r3f-844c-mc37  Medium    0.4% (60th)    0.2
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.7.0                              go-module  GHSA-vvpx-j8f3-3w6h  High      0.3% (49th)    0.2
github.com/gin-gonic/gin    v1.8.1                              1.9.1                              go-module  GHSA-2c4m-59x9-fr2g  Medium    0.4% (61st)    0.2
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.4.0                              go-module  GHSA-xrjj-mj9h-534m  Medium    0.3% (55th)    0.2
github.com/gin-gonic/gin    v1.8.1                              1.9.0                              go-module  GHSA-3vp4-m3rf-835h  Medium    0.2% (46th)    0.1
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.17.0                             go-module  GHSA-4374-p667-p6c8  High      0.2% (36th)    0.1
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.0.0-20210520170846-37e1c6afe023  go-module  GHSA-83g2-8m93-v3w7  High      0.1% (33rd)    0.1
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.0.0-20220906165146-f3363e06e74c  go-module  GHSA-69cg-p879-7622  High      0.1% (32nd)    < 0.1
golang.org/x/sys            v0.0.0-20210806184541-e5e7981a1069  0.0.0-20220412211240-33da011f77ad  go-module  GHSA-p782-xgp4-8hr8  Medium    0.2% (39th)    < 0.1
golang.org/x/crypto         v0.0.0-20210711020723-a769d52b0f97  0.0.0-20220314234659-1baeb1ce4c0b  go-module  GHSA-8c26-wmh5-6g9v  High      < 0.1% (25th)  < 0.1
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.38.0                             go-module  GHSA-vvgc-356p-c3xw  Medium    0.1% (30th)    < 0.1
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.13.0                             go-module  GHSA-2wrh-6pvc-2jm9  Medium    < 0.1% (26th)  < 0.1
golang.org/x/text           v0.3.6                              0.3.8                              go-module  GHSA-69ch-w2m2-3vjp  High      < 0.1% (16th)  < 0.1
golang.org/x/text           v0.3.6                              0.3.7                              go-module  GHSA-ppp9-7jff-5vj2  High      < 0.1% (16th)  < 0.1
golang.org/x/crypto         v0.0.0-20210711020723-a769d52b0f97  0.0.0-20211202192323-5770296d904e  go-module  GHSA-gwc9-m7rh-j2ww  High      < 0.1% (9th)   < 0.1
golang.org/x/crypto         v0.0.0-20210711020723-a769d52b0f97  0.45.0                             go-module  GHSA-j5w8-q4qc-rx2x  Medium    < 0.1% (13th)  < 0.1
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.0.0-20210428140749-89ef3d95e781  go-module  GHSA-h86h-8ppg-mxmh  Medium    < 0.1% (6th)   < 0.1
golang.org/x/net            v0.0.0-20210226172049-e18ecbb05110  0.36.0                             go-module  GHSA-qxp5-gwg8-xv66  Medium    < 0.1% (6th)   < 0.1
golang.org/x/crypto         v0.0.0-20210711020723-a769d52b0f97  0.45.0                             go-module  GHSA-f6x5-jh6r-wrfv  Medium    < 0.1% (2nd)   < 0.1
```

The only difference is the three issues in apk modules, which the multistage build does not have since it uses a scratch image.  Otherwise, the 24 issues in go modules are all fixed. One is particularly worrysome since it is in the Known Exploited Vulnerabilities catalog.

## 3. Remediation

### Patch the Go application (and its dependencies)

```
$ go get -u ./...
go: upgraded go 1.18 => 1.25.0
go: added github.com/bytedance/gopkg v0.1.4
go: added github.com/bytedance/sonic v1.15.1
go: added github.com/bytedance/sonic/loader v0.5.1
go: added github.com/cloudwego/base64x v0.1.7
go: added github.com/gabriel-vasile/mimetype v1.4.13
go: upgraded github.com/gin-contrib/sse v0.1.0 => v1.1.1
go: upgraded github.com/gin-gonic/gin v1.8.1 => v1.12.0
go: upgraded github.com/go-playground/locales v0.14.0 => v0.14.1
go: upgraded github.com/go-playground/universal-translator v0.18.0 => v0.18.1
go: upgraded github.com/go-playground/validator/v10 v10.10.0 => v10.30.2
go: upgraded github.com/goccy/go-json v0.9.7 => v0.10.6
go: added github.com/goccy/go-yaml v1.19.2
go: added github.com/klauspost/cpuid/v2 v2.3.0
go: upgraded github.com/leodido/go-urn v1.2.1 => v1.4.0
go: upgraded github.com/mattn/go-isatty v0.0.14 => v0.0.22
go: upgraded github.com/modern-go/concurrent v0.0.0-20180228061459-e0a39a4cb421 => v0.0.0-20180306012644-bacd9c7ef1dd
go: upgraded github.com/pelletier/go-toml/v2 v2.0.1 => v2.3.1
go: added github.com/quic-go/qpack v0.6.0
go: added github.com/quic-go/quic-go v0.59.0
go: added github.com/twitchyliquid64/golang-asm v0.15.1
go: upgraded github.com/ugorji/go/codec v1.2.7 => v1.3.1
go: added go.mongodb.org/mongo-driver/v2 v2.6.0
go: added golang.org/x/arch v0.26.0
go: upgraded golang.org/x/crypto v0.0.0-20210711020723-a769d52b0f97 => v0.50.0
go: upgraded golang.org/x/net v0.0.0-20210226172049-e18ecbb05110 => v0.53.0
go: upgraded golang.org/x/sys v0.0.0-20210806184541-e5e7981a1069 => v0.43.0
go: upgraded golang.org/x/text v0.3.6 => v0.36.0
go: upgraded google.golang.org/protobuf v1.28.0 => v1.36.11

$ go mod tidy
```

### Update or optimize base images as needed

```
$ cat Dockerfile-single-patched
FROM cgr.dev/chainguard/go:latest-dev
WORKDIR /app
COPY . .
RUN go build -o hello-server .
EXPOSE 8080
ENTRYPOINT ["./hello-server"]

$ docker build -t host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-patched -f Dockerfile-single-patched .

$ cat Dockerfile-multi-patched
FROM cgr.dev/chainguard/go:latest AS builder
WORKDIR /app
COPY . .
RUN go build -o hello-server .

FROM cgr.dev/chainguard/go:latest
COPY --from=builder /app/hello-server /hello-server
EXPOSE 8080
ENTRYPOINT ["/hello-server"]
```

### Rescan and document improvements

```
$ grype host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-patched
 ✔ Loaded image                            host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-patched
 ✔ Parsed image                              sha256:8c5b82799cf13d7199fe67c37975e68dbbc88c6dbc88be802aedd93fddce6ef6
 ✔ Cataloged contents                               6156d5ad2ee5683c3679f38428432197e3351587b29e60b90313cf9117355f4f
   ├── ✔ Packages                        [97 packages]
   ├── ✔ Executables                     [254 executables]
   ├── ✔ File metadata                   [10,100 locations]
   └── ✔ File digests                    [10,100 files]
 ✔ Scanned for vulnerabilities     [0 vulnerability matches]
   ├── by severity: 0 critical, 0 high, 0 medium, 0 low, 0 negligible
   └── by status:   0 fixed, 0 not-fixed, 0 ignored
No vulnerabilities found

$ grype host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-patched
 ✔ Loaded image                             host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-patched
 ✔ Parsed image                              sha256:8ef8067b3a97e5ff83cf392c84cb1208a0c251a62701232398ea300d71f026d7
 ✔ Cataloged contents                               12e207943f1b7e7a0860715fe7af977d5411efc7180e30a17884540a0536a174
   ├── ✔ Packages                        [95 packages]
   ├── ✔ Executables                     [250 executables]
   ├── ✔ File metadata                   [10,093 locations]
   └── ✔ File digests                    [10,093 files]
 ✔ Scanned for vulnerabilities     [0 vulnerability matches]
   ├── by severity: 0 critical, 0 high, 0 medium, 0 low, 0 negligible
   └── by status:   0 fixed, 0 not-fixed, 0 ignored
No vulnerabilities found
```

No vulnerabilities found.  Updating the go dependencies fixed the 24 issues in the go modules.  In the single-stage image, switching to the chainguard go:latest-dev image removed the apk issues.  In the multi-stage image, using the go:latest image as the final stage sidesteps those issues.

## 4. Supply Chain Security

### Generate SBOM for each container

```
$ syft -o spdx-json=single-unpatched-spdx.json -o cyclonedx-json=single-unpatched-cdx.json host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-unpatched
 ✔ Loaded image                          host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-unpatched
 ✔ Parsed image                              sha256:6b56f658ddb4ceea03e5ef9a4d7161ae69c0da32e9b59315400706f485cdf41c
 ✔ Cataloged contents                               863259eb0b6f90af8b7b5f84ea8fee81e1d929fd92fe94767edd4e88e7dbc0f8
   ├── ✔ Packages                        [55 packages]
   ├── ✔ Executables                     [58 executables]
   ├── ✔ File metadata                   [240 locations]

$ syft -o spdx-json=multi-unpatched-spdx.json -o cyclonedx-json=multi-unpatched-cdx.json host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-unpatched
 ✔ Loaded image                           host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-unpatched
 ✔ Parsed image                              sha256:ed3b39a5e53520a73dc8f5f850de449abe1ae0805eceb88b6411ef46194ebbc2
 ✔ Cataloged contents                               e77b47f63d7574c27545b23c455ae28bc0a63d6b04da7e4a69416aefd2f630d1
   ├── ✔ Packages                        [17 packages]
   ├── ✔ Executables                     [1 executables]
   ├── ✔ File digests                    [1 files]

$ syft -o spdx-json=single-patched-spdx.json -o cyclonedx-json=single-patched-cdx.json host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-patched
 ✔ Loaded image                            host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-patched
 ✔ Parsed image                              sha256:8c5b82799cf13d7199fe67c37975e68dbbc88c6dbc88be802aedd93fddce6ef6
 ✔ Cataloged contents                               6156d5ad2ee5683c3679f38428432197e3351587b29e60b90313cf9117355f4f
   ├── ✔ Packages                        [111 packages]
   ├── ✔ Executables                     [254 executables]
   ├── ✔ File metadata                   [10,103 locations]

$ syft -o spdx-json=multi-patched-spdx.json -o cyclonedx-json=multi-patched-cdx.json host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-patched
 ✔ Loaded image                             host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-patched
 ✔ Parsed image                              sha256:8ef8067b3a97e5ff83cf392c84cb1208a0c251a62701232398ea300d71f026d7
 ✔ Cataloged contents                               12e207943f1b7e7a0860715fe7af977d5411efc7180e30a17884540a0536a174
   ├── ✔ Packages                        [106 packages]
   ├── ✔ Executables                     [250 executables]
   ├── ✔ File metadata                   [10,093 locations]
```

### Sign all container images
### Push signed images to your local registry

```
$ docker push host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-unpatched
The push refers to repository [host.docker.internal:5555/pvnovarese/hello-melange-apko-go]
4f4fb700ef54: Mounted from pvnovarese/hello-melange-apk
1a70bdedd442: Pushed
1d5c7638f979: Pushed
2c4e3db30483: Pushed
07cc3f3d99a3: Pushed
17423c887b37: Pushed
6a0ac1617861: Mounted from pvnovarese/hello-melange-apk
f306aef94490: Pushed
1512b43389a8: Pushed
single-unpatched: digest: sha256:6fffa0db0870770662f3e756a432f8d8cca50468a2482b83f751329a46bd5145 size: 856

$ cosign sign localhost:5555/pvnovarese/hello-melange-apko-go:single-unpatched@sha256:6fffa0db0870770662f3e756a432f8d8cca50468a2482b83f751329a46bd5145
Generating ephemeral keys...
[...snipped...]

$ docker push host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-unpatched
The push refers to repository [host.docker.internal:5555/pvnovarese/hello-melange-apko-go]
55616a861f59: Pushed
88545892ff2a: Pushed
35e08dadb549: Mounted from pvnovarese/hello-melange-apk
multi-unpatched: digest: sha256:48be2d77720e3616f81f7d18f595d08ce7f6260a4d6870a8634b61fdea25823e size: 855

$ cosign sign localhost:5555/pvnovarese/hello-melange-apko-go:multi-unpatched@sha256:48be2d77720e3616f81f7d18f595d08ce7f6260a4d6870a8634b61fdea25823e
Generating ephemeral keys...
[...snipped...]

$ docker push host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-patched
The push refers to repository [host.docker.internal:5555/pvnovarese/hello-melange-apko-go]
19bc0a29fc10: Pushed
7d4ff56720d7: Pushed
e59fd7968801: Pushed
2c39adb5c5c0: Pushed
c80080368687: Pushed
9c760148b2ff: Pushed
41537debae32: Pushed
85c92b47e0a8: Pushed
84066de16370: Pushed
f1a66d5f775d: Pushed
a9acbfab50a5: Pushed
a2abcdcd349b: Pushed
f54b384c0cd6: Pushed
44febfb5b2d2: Pushed
5f12258674dc: Pushed
single-patched: digest: sha256:7de8207e1120e4ae3331c287df56b407c08b8f96167eccf888074316b2f83e72 size: 856

$ cosign sign localhost:5555/pvnovarese/hello-melange-apko-go:single-patched@sha256:7de8207e1120e4ae3331c287df56b407c08b8f96167eccf888074316b2f83e72
Generating ephemeral keys...
[...snipped...]

$ docker push host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-patched
The push refers to repository [host.docker.internal:5555/pvnovarese/hello-melange-apko-go]
f54b384c0cd6: Layer already exists
a9acbfab50a5: Layer already exists
5f12258674dc: Layer already exists
e59fd7968801: Layer already exists
19bc0a29fc10: Layer already exists
95dcb017dab9: Pushed
9371113a7275: Pushed
c80080368687: Layer already exists
44febfb5b2d2: Layer already exists
2c39adb5c5c0: Layer already exists
a2abcdcd349b: Layer already exists
7603be13c26a: Pushed
750ea1a3a123: Pushed
multi-patched: digest: sha256:7aa6946d6f01bf4eca843264e23a61f0bbb1baa97a2244877c325b61c073fdf3 size: 856

$ cosign sign localhost:5555/pvnovarese/hello-melange-apko-go:multi-patched@sha256:7aa6946d6f01bf4eca843264e23a61f0bbb1baa97a2244877c325b61c073fdf3
Generating ephemeral keys...
[...snipped...]

$ cosign verify localhost:5555/pvnovarese/hello-melange-apko-go@sha256:6fffa0db0870770662f3e756a432f8d8cca50468a2482b83f751329a46bd5145 --certificate-identity=pvn@novarese.net --certificate-oidc-issuer=https://github.com/login/oauth

Verification for localhost:5555/pvnovarese/hello-melange-apko-go@sha256:6fffa0db0870770662f3e756a432f8d8cca50468a2482b83f751329a46bd5145 --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - Existence of the claims in the transparency log was verified offline
  - The code-signing certificate was verified using trusted certificate authority certificates

[{"critical":{"identity":{"docker-reference":"localhost:5555/pvnovarese/hello-melange-apko-go@sha256:6fffa0db0870770662f3e756a432f8d8cca50468a2482b83f751329a46bd5145"},"image":{"docker-manifest-digest":"sha256:6fffa0db0870770662f3e756a432f8d8cca50468a2482b83f751329a46bd5145"},"type":"https://sigstore.dev/cosign/sign/v1"},"optional":{}}]

$ cosign verify localhost:5555/pvnovarese/hello-melange-apko-go@sha256:48be2d77720e3616f81f7d18f595d08ce7f6260a4d6870a8634b61fdea25823e --certificate-identity=pvn@novarese.net --certificate-oidc-issuer=https://github.com/login/oauth

Verification for localhost:5555/pvnovarese/hello-melange-apko-go@sha256:48be2d77720e3616f81f7d18f595d08ce7f6260a4d6870a8634b61fdea25823e --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - Existence of the claims in the transparency log was verified offline
  - The code-signing certificate was verified using trusted certificate authority certificates

[{"critical":{"identity":{"docker-reference":"localhost:5555/pvnovarese/hello-melange-apko-go@sha256:48be2d77720e3616f81f7d18f595d08ce7f6260a4d6870a8634b61fdea25823e"},"image":{"docker-manifest-digest":"sha256:48be2d77720e3616f81f7d18f595d08ce7f6260a4d6870a8634b61fdea25823e"},"type":"https://sigstore.dev/cosign/sign/v1"},"optional":{}}]

$ cosign verify localhost:5555/pvnovarese/hello-melange-apko-go@sha256:7de8207e1120e4ae3331c287df56b407c08b8f96167eccf888074316b2f83e72 --certificate-identity=pvn@novarese.net --certificate-oidc-issuer=https://github.com/login/oauth

Verification for localhost:5555/pvnovarese/hello-melange-apko-go@sha256:7de8207e1120e4ae3331c287df56b407c08b8f96167eccf888074316b2f83e72 --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - Existence of the claims in the transparency log was verified offline
  - The code-signing certificate was verified using trusted certificate authority certificates

[{"critical":{"identity":{"docker-reference":"localhost:5555/pvnovarese/hello-melange-apko-go@sha256:7de8207e1120e4ae3331c287df56b407c08b8f96167eccf888074316b2f83e72"},"image":{"docker-manifest-digest":"sha256:7de8207e1120e4ae3331c287df56b407c08b8f96167eccf888074316b2f83e72"},"type":"https://sigstore.dev/cosign/sign/v1"},"optional":{}}]

$ cosign verify localhost:5555/pvnovarese/hello-melange-apko-go@sha256:7aa6946d6f01bf4eca843264e23a61f0bbb1baa97a2244877c325b61c073fdf3 --certificate-identity=pvn@novarese.net --certificate-oidc-issuer=https://github.com/login/oauth

Verification for localhost:5555/pvnovarese/hello-melange-apko-go@sha256:7aa6946d6f01bf4eca843264e23a61f0bbb1baa97a2244877c325b61c073fdf3 --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - Existence of the claims in the transparency log was verified offline
  - The code-signing certificate was verified using trusted certificate authority certificates

[{"critical":{"identity":{"docker-reference":"localhost:5555/pvnovarese/hello-melange-apko-go@sha256:7aa6946d6f01bf4eca843264e23a61f0bbb1baa97a2244877c325b61c073fdf3"},"image":{"docker-manifest-digest":"sha256:7aa6946d6f01bf4eca843264e23a61f0bbb1baa97a2244877c325b61c073fdf3"},"type":"https://sigstore.dev/cosign/sign/v1"},"optional":{}}]
```

## 5. Deployment

### Deploy as a standalone container

```
$ docker run -p 8080:8080 --rm host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-unpatched
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /                         --> main.main.func1 (3 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.
[GIN-debug] Environment variable PORT is undefined. Using port :8080 by default
[GIN-debug] Listening and serving HTTP on :8080
[GIN] 2026/05/07 - 21:06:59 | 200 |     828.118µs | 142.250.113.141 | GET      "/"

$ curl localhost:8080
Hello World!

$ docker run -p 8080:8080 --rm host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-unpatched
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /                         --> main.main.func1 (3 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.
[GIN-debug] Environment variable PORT is undefined. Using port :8080 by default
[GIN-debug] Listening and serving HTTP on :8080
[GIN] 2026/05/07 - 21:07:46 | 200 |    1.477777ms | 142.250.113.141 | GET      "/"

$ curl localhost:8080
Hello World!

$ docker run -p 8080:8080 --rm host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-patched
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /                         --> main.main.func1 (3 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://github.com/gin-gonic/gin/blob/master/docs/doc.md#dont-trust-all-proxies for details.
[GIN-debug] Environment variable PORT is undefined. Using port :8080 by default
[GIN-debug] Listening and serving HTTP on :8080
[GIN] 2026/05/07 - 21:08:14 | 200 |  37.65µs | 142.250.113.141 | GET      "/"

$ curl localhost:8080
Hello World!

$ docker run -p 8080:8080 --rm host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-patched
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /                         --> main.main.func1 (3 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://github.com/gin-gonic/gin/blob/master/docs/doc.md#dont-trust-all-proxies for details.
[GIN-debug] Environment variable PORT is undefined. Using port :8080 by default
[GIN-debug] Listening and serving HTTP on :8080
[GIN] 2026/05/07 - 21:08:50 | 200 | 129.396µs | 142.250.113.141 | GET      "/"

$ curl localhost:8080
Hello World!
```

### Deploy to your Kubernetes cluster

set up to verify image signatures:
```
$ helm repo add sigstore https://sigstore.github.io/helm-charts
"sigstore" has been added to your repositories

$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "sigstore" chart repository
Update Complete. ⎈Happy Helming!⎈

$ helm install policy-controller sigstore/policy-controller \
  --namespace cosign-system \
  --create-namespace
NAME: policy-controller
LAST DEPLOYED: Thu May  7 13:05:38 2026
NAMESPACE: cosign-system
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE: None

$ cat namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: default
  labels:
    policy.sigstore.dev/include: "true"

$ kubectl apply -f namespace.yaml
namespace/default configured

$ cat clusterimagepolicy.yaml
apiVersion: policy.sigstore.dev/v1beta1
kind: ClusterImagePolicy
metadata:
  name: verify-hello-melange-apk
spec:
  images:
    - glob: "host.docker.internal:5555/pvnovarese/hello-melange-apk-go*"
  authorities:
    - keyless:
        url: https://fulcio.sigstore.dev
        identities:
          - subjectRegExp: "pvn@novarese.net"
            issuer: "https://github.com/login/oauth"

$ kubectl apply -f clusterimagepolicy.yaml
clusterimagepolicy.policy.sigstore.dev/verify-hello-melange-apk created
```

now for deployments:

First, the single-stage unpatched image:
```
$ cat deployment-single-unpatched.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-melange-apk
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-melange-apk
  template:
    metadata:
      labels:
        app: hello-melange-apk
    spec:
      containers:
        - name: hello-server
          image: host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-unpatched@sha256:6fffa0db0870770662f3e756a432f8d8cca50468a2482b83f751329a46bd5145
          ports:
            - containerPort: 8080
          securityContext:
            runAsNonRoot: true
            runAsUser: 65532
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL

$ k apply -f deployment-single-unpatched.yaml
deployment.apps/hello-melange-apk created

$ k get pods
NAME                                READY   STATUS    RESTARTS   AGE
hello-melange-apk-c79b96f98-br5bf   1/1     Running   0          70s

$ kubectl port-forward deployment/hello-melange-apk 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080

$ curl localhost:8080
Hello World!

$ k delete deployment hello-melange-apk
deployment.apps "hello-melange-apk" deleted from default namespace
```

next, the multi-stage unpatched image:

```
$ cat deployment-multi-unpatched.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-melange-apk
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-melange-apk
  template:
    metadata:
      labels:
        app: hello-melange-apk
    spec:
      containers:
        - name: hello-server
          image: host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-unpatched@sha256:48be2d77720e3616f81f7d18f595d08ce7f6260a4d6870a8634b61fdea25823e
          ports:
            - containerPort: 8080
          securityContext:
            runAsNonRoot: true
            runAsUser: 65532
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL

$ k apply -f deployment-multi-unpatched.yaml
deployment.apps/hello-melange-apk created

$ k get pods
NAME                                 READY   STATUS    RESTARTS   AGE
hello-melange-apk-5645548698-kq89r   1/1     Running   0          4s

$ kubectl port-forward deployment/hello-melange-apk 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080

$ curl localhost:8080
Hello World!

$ k delete deployment hello-melange-apk
deployment.apps "hello-melange-apk" deleted from default namespace
```

next, single-stage patched image:

```
$ cat deployment-single-patched.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-melange-apk
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-melange-apk
  template:
    metadata:
      labels:
        app: hello-melange-apk
    spec:
      containers:
        - name: hello-server
          image: host.docker.internal:5555/pvnovarese/hello-melange-apko-go:single-patched@sha256:7de8207e1120e4ae3331c287df56b407c08b8f96167eccf888074316b2f83e72
          ports:
            - containerPort: 8080
          securityContext:
            runAsNonRoot: true
            runAsUser: 65532
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL

$ k apply -f deployment-single-patched.yaml
deployment.apps/hello-melange-apk created

$ k get pods
NAME                                 READY   STATUS    RESTARTS   AGE
hello-melange-apk-6f484897c4-d4vqq   1/1     Running   0          39s

$ kubectl port-forward deployment/hello-melange-apk 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080

$ curl localhost:8080
Hello World!

$ k delete deployment hello-melange-apk
deployment.apps "hello-melange-apk" deleted from default namespace
```

finally, the multi-stage patched image:

```
$ cat deployment-multi-patched.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-melange-apk
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-melange-apk
  template:
    metadata:
      labels:
        app: hello-melange-apk
    spec:
      containers:
        - name: hello-server
          image: host.docker.internal:5555/pvnovarese/hello-melange-apko-go:multi-patched@sha256:7aa6946d6f01bf4eca843264e23a61f0bbb1baa97a2244877c325b61c073fdf3
          ports:
            - containerPort: 8080
          securityContext:
            runAsNonRoot: true
            runAsUser: 65532
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
$ k apply -f deployment-multi-patched.yaml
deployment.apps/hello-melange-apk created

$ k get pods
NAME                                 READY   STATUS    RESTARTS   AGE
hello-melange-apk-7664778f54-6h5ws   1/1     Running   0          22s

$ kubectl port-forward deployment/hello-melange-apk 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080

$ curl localhost:8080
Hello World!
```

### Validate both deployments are functional

see above

## Deliverables

### All Dockerfiles

in this directory

### Vulnerability scan reports (before/after)

in the `vuln_reports` directory

### SBOM files

in the `sbom` directory

### Kubernetes manifests

in the `manifests` directory

### Documentation of findings and methodology

in this file.
