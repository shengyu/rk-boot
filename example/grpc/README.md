# rk-grpc
Interceptor & bootstrapper designed for grpc. Currently, supports bellow functionalities.

| Name | Description |
| ---- | ---- |
| Start with YAML | Start service with YAML config. |
| Start with code | Start service from code. |
| GRPC Service | GRPC service defined with protocol buffer. |
| GRPC Gateway Service | GRPC Gateway service with new port. |
| Swagger Service | Swagger UI with same port as GRPC Gateway. |
| Common Service | List of common API available on GRPC, GRPC Gateway and swagger. |
| TV Service | A Web UI shows application and environment information. |
| Metrics interceptor | Collect RPC metrics and export as prometheus client with same port of GRPC gateway. |
| Static file handler | A Web UI shows files could be downloaded from server, currently support source of local and pkger. |
| Log interceptor | Log every RPC requests as event with rk-query. |
| Trace interceptor | Collect RPC trace and export it to stdout, file or jaeger. |
| Panic interceptor | Recover from panic for RPC requests and log it. |
| Meta interceptor | Send application metadata as header to client and GRPC Gateway. |
| Auth interceptor | Support [Basic Auth], [Bearer Token] and [API Key] authrization types. |
| RateLimit interceptor | Limit request rate from interceptor. |
| Timeout interceptor | Timing out request based on configuration. |
| CORS interceptor | CORS interceptor for grpc-gateway. |
| JWT interceptor | JWT interceptor on server side. |
| Secure interceptor | Secure interceptor for grpc-gateway. |
| CSRF interceptor | CSRF interceptor for grpc-gateway. |

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Installation](#installation)
- [Quick start](#quick-start)
- [YAML Config](#yaml-config)
  - [GRPC Service](#grpc-service)
  - [Common Service](#common-service)
  - [GRPC Gateway Service](#grpc-gateway-service)
  - [Prom Client](#prom-client)
  - [TV Service](#tv-service)
  - [Static file handler Service](#static-file-handler-service)
  - [Swagger Service](#swagger-service)
  - [Interceptors](#interceptors)
    - [Log](#log)
    - [Metrics](#metrics)
    - [Auth](#auth)
    - [Meta](#meta)
    - [Tracing](#tracing)
    - [RateLimit](#ratelimit)
    - [Timeout](#timeout)
    - [CORS](#cors)
    - [JWT](#jwt)
    - [Secure](#secure)
    - [CSRF](#csrf)
  - [Development Status: Stable](#development-status-stable)
  - [Appendix](#appendix)
  - [Contributing](#contributing)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Installation
`go get github.com/rookie-ninja/rk-boot`

## Quick start
- boot.yaml
```yaml
---
grpc:
  - name: greeter             # Required, Name of grpc entry
    port: 8080                # Required, Port of grpc entry
    enabled: true             # Required, Enable grpc entry
    commonService:
      enabled: true           # Optional, Enable common service
    tv:
      enabled: true           # Optional, Enable RK TV
    sw:
      enabled: true           # Optional, Enable Swagger UI
```
- main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```
```shell script
$ go run main.go
$ curl -X GET localhost:8080/rk/v1/healthy
{"healthy":true}
```
- Swagger: http://localhost:8080/sw

![grpc-sw](../../img/grpc-sw.png)

- TV: http://localhost:8080/rk/v1/tv

![grpc-tv](../../img/grpc-tv.png)

## YAML Config
Available configuration
User can start multiple grpc servers at the same time. Please make sure use different port and name.

### GRPC Service
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.name | The name of grpc server | string | N/A |
| grpc.port | The port of grpc server | integer | nil, server won't start |
| grpc.enabled | Enable grpc entry | bool | false |
| grpc.description | Description of grpc entry. | string | "" |
| grpc.enableReflection | Enable grpc server reflection | boolean | false |
| grpc.enableRkGwOption | Enable RK style gateway server options. [detail](boot/gw_server_options.go) | false |
| grpc.gwMappingFilePaths | The grpc gateway mapping file path. [example](boot/api/v1/gw_mapping.yaml) | string array | [] |
| grpc.cert.ref | Reference of cert entry declared in [cert entry](https://github.com/rookie-ninja/rk-entry#certentry) | string | "" |
| grpc.logger.zapLogger.ref | Reference of zapLoggerEntry declared in [zapLoggerEntry](https://github.com/rookie-ninja/rk-entry#zaploggerentry) | string | "" |
| grpc.logger.eventLogger.ref | Reference of eventLoggerEntry declared in [eventLoggerEntry](https://github.com/rookie-ninja/rk-entry#eventloggerentry) | string | "" |

### Common Service
```yaml
http:
  rules:
    - selector: rk.api.v1.RkCommonService.Healthy
      get: /rk/v1/healthy
    - selector: rk.api.v1.RkCommonService.Gc
      get: /rk/v1/gc
    - selector: rk.api.v1.RkCommonService.Info
      get: /rk/v1/info
    - selector: rk.api.v1.RkCommonService.Configs
      get: /rk/v1/configs
    - selector: rk.api.v1.RkCommonService.Apis
      get: /rk/v1/apis
    - selector: rk.api.v1.RkCommonService.Sys
      get: /rk/v1/sys
    - selector: rk.api.v1.RkCommonService.Req
      get: /rk/v1/req
    - selector: rk.api.v1.RkCommonService.Entries
      get: /rk/v1/entries
    - selector: rk.api.v1.RkCommonService.Certs
      get: /rk/v1/certs
    - selector: rk.api.v1.RkCommonService.Logs
      get: /rk/v1/logs
    - selector: rk.api.v1.RkCommonService.Deps
      get: /rk/v1/deps
    - selector: rk.api.v1.RkCommonService.License
      get: /rk/v1/license
    - selector: rk.api.v1.RkCommonService.Readme
      get: /rk/v1/readme
    - selector: rk.api.v1.RkCommonService.Git
      get: /rk/v1/git
    - selector: rk.api.v1.RkCommonService.GwErrorMapping
      get: /rk/v1/gwErrorMapping
```

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.commonService.enabled | Enable embedded common service | boolean | false |

### GRPC Gateway Service
By default, gateway will always be started with grpc with same port. 

Please refer to bellow repository for detailed explanations.
- [protobuf-go/encoding/protojson/encode.go](https://github.com/protocolbuffers/protobuf-go/blob/master/encoding/protojson/encode.go#L43)
- [protobuf-go/encoding/protojson/decode.go ](https://github.com/protocolbuffers/protobuf-go/blob/master/encoding/protojson/decode.go#L33)

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.gwOption.marshal.multiline | Enable multiline in grpc-gateway marshaller | bool | false |
| grpc.gwOption.marshal.emitUnpopulated | Enable emitUnpopulated in grpc-gateway marshaller | bool | false |
| grpc.gwOption.marshal.indent | Set indent in grpc-gateway marshaller | string | "  " |
| grpc.gwOption.marshal.allowPartial | Enable allowPartial in grpc-gateway marshaller | bool | false |
| grpc.gwOption.marshal.useProtoNames | Enable useProtoNames in grpc-gateway marshaller | bool | false |
| grpc.gwOption.marshal.useEnumNumbers | Enable useEnumNumbers in grpc-gateway marshaller | bool | false |
| grpc.gwOption.unmarshal.allowPartial | Enable allowPartial in grpc-gateway unmarshaler | bool | false |
| grpc.gwOption.unmarshal.discardUnknown | Enable discardUnknown in grpc-gateway unmarshaler | bool | false |

### Prom Client
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.prom.enabled | Enable prometheus | boolean | false |
| grpc.prom.path | Path of prometheus | string | /metrics |
| grpc.prom.pusher.enabled | Enable prometheus pusher | bool | false |
| grpc.prom.pusher.jobName | Job name would be attached as label while pushing to remote pushgateway | string | "" |
| grpc.prom.pusher.remoteAddress | PushGateWay address, could be form of http://x.x.x.x or x.x.x.x | string | "" |
| grpc.prom.pusher.intervalMs | Push interval in milliseconds | string | 1000 |
| grpc.prom.pusher.basicAuth | Basic auth used to interact with remote pushgateway, form of [user:pass] | string | "" |
| grpc.prom.pusher.cert.ref | Reference of rkentry.CertEntry | string | "" |

### TV Service
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.tv.enabled | Enable RK TV | boolean | false |

### Static file handler Service
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.static.enabled | Optional, Enable static file handler | boolean | false |
| grpc.static.path | Optional, path of static file handler | string | /rk/v1/static |
| grpc.static.sourceType | Required, local and pkger supported | string | "" |
| grpc.static.sourcePath | Required, full path of source directory | string | "" |

- About [pkger](https://github.com/markbates/pkger)
User can use pkger command line tool to embed static files into .go files.

Please use sourcePath like: github.com/rookie-ninja/rk-grpc:/boot/assets

### Swagger Service
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.sw.enabled | Enable swagger service over gRpc server | boolean | false |
| grpc.sw.path | The path access swagger service from web | string | /sw |
| grpc.sw.jsonPath | Where the swagger.json files are stored locally | string | "" |
| grpc.sw.headers | Headers would be sent to caller as scheme of [key:value] | []string | [] |

### Interceptors
#### Log
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.loggingZap.enabled | Enable log interceptor | boolean | false |

We will log two types of log for every RPC call.
- zapLogger

Contains user printed logging with requestId or traceId.

- eventLogger

Contains per RPC metadata, response information, environment information and etc.

| Field | Description |
| ---- | ---- |
| endTime | As name described |
| startTime | As name described |
| elapsedNano | Elapsed time for RPC in nanoseconds |
| timezone | As name described |
| ids | Contains three different ids(eventId, requestId and traceId). If meta interceptor was enabled or event.SetRequestId() was called by user, then requestId would be attached. eventId would be the same as requestId if meta interceptor was enabled. If trace interceptor was enabled, then traceId would be attached. |
| app | Contains [appName, appVersion](https://github.com/rookie-ninja/rk-entry#appinfoentry), entryName, entryType. |
| env | Contains arch, az, domain, hostname, localIP, os, realm, region. realm, region, az, domain were retrieved from environment variable named as REALM, REGION, AZ and DOMAIN. "*" means empty environment variable.|
| payloads | Contains RPC related metadata |
| error | Contains errors if occur |
| counters | Set by calling event.SetCounter() by user. |
| pairs | Set by calling event.AddPair() by user. |
| timing | Set by calling event.StartTimer() and event.EndTimer() by user. |
| remoteAddr |  As name described |
| operation | RPC method name |
| resCode | Response code of RPC |
| eventStatus | Ended or InProgress |

- example

```shell script
------------------------------------------------------------------------
endTime=2021-06-24T05:58:48.282193+08:00
startTime=2021-06-24T05:58:48.28204+08:00
elapsedNano=153005
timezone=CST
ids={"eventId":"573ce6a8-308b-4fc0-9255-33608b9e41d4","requestId":"573ce6a8-308b-4fc0-9255-33608b9e41d4"}
app={"appName":"rk-boot","appVersion":"master-xxx","entryName":"greeter","entryType":"GrpcEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.6","os":"darwin","realm":"*","region":"*"}
payloads={"grpcMethod":"Healthy","grpcService":"rk.api.v1.RkCommonService","grpcType":"unaryServer","gwMethod":"GET","gwPath":"/rk/v1/healthy","gwScheme":"http","gwUserAgent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.77 Safari/537.36"}
error={}
counters={}
pairs={"healthy":"true"}
timing={}
remoteAddr=localhost:57135
operation=/rk.api.v1.RkCommonService/Healthy
resCode=OK
eventStatus=Ended
EOE
```

#### Metrics
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.metricsProm.enabled | Enable metrics interceptor | boolean | false |

#### Auth
Enable the server side auth. codes.Unauthenticated would be returned to client if not authorized with user defined credential.

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.auth.enabled | Enable auth interceptor | boolean | false |
| grpc.interceptors.auth.basic | Basic auth credentials as scheme of <user:pass> | []string | [] |
| grpc.interceptors.auth.bearer | Bearer auth tokens | []string | [] |
| grpc.interceptors.auth.api | API key | []string | [] |

#### Meta
Send application metadata as header to client and GRPC Gateway.

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.meta.enabled | Enable meta interceptor | boolean | false |
| grpc.interceptors.meta.prefix | Header key was formed as X-<Prefix>-XXX | string | RK |

#### Tracing
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.tracingTelemetry.enabled | Enable tracing interceptor | boolean | false |
| grpc.interceptors.tracingTelemetry.exporter.file.enabled | Enable file exporter | boolean | RK |
| grpc.interceptors.tracingTelemetry.exporter.file.outputPath | Export tracing info to files | string | stdout |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.agent.enabled | Export tracing info to jaeger agent | boolean | false |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.agent.host | As name described | string | localhost |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.agent.port | As name described | int | 6831 |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.collector.enabled | Export tracing info to jaeger collector | boolean | false |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.collector.endpoint | As name described | string | http://localhost:16368/api/trace |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.collector.username | As name described | string | "" |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.collector.password | As name described | string | "" |

#### RateLimit
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.rateLimit.enabled | Enable rate limit interceptor | boolean | false |
| grpc.interceptors.rateLimit.algorithm | Provide algorithm, tokenBucket and leakyBucket are available options | string | tokenBucket |
| grpc.interceptors.rateLimit.reqPerSec | Request per second globally | int | 0 |
| grpc.interceptors.rateLimit.paths.path | gRPC full name | string | "" |
| grpc.interceptors.rateLimit.paths.reqPerSec | Request per second by gRPC full method name | int | 0 |

#### Timeout
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.timeout.enabled | Enable timeout interceptor | boolean | false |
| grpc.interceptors.timeout.timeoutMs | Global timeout in milliseconds. | int | 5000 |
| grpc.interceptors.timeout.paths.path | Full path | string | "" |
| grpc.interceptors.timeout.paths.timeoutMs | Timeout in milliseconds by full path | int | 5000 |

#### CORS
Interceptor for grpc-gateway.

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.cors.enabled | Enable cors interceptor | boolean | false |
| grpc.interceptors.cors.allowOrigins | Provide allowed origins with wildcard enabled. | []string | * |
| grpc.interceptors.cors.allowMethods | Provide allowed methods returns as response header of OPTIONS request. | []string | All http methods |
| grpc.interceptors.cors.allowHeaders | Provide allowed headers returns as response header of OPTIONS request. | []string | Headers from request |
| grpc.interceptors.cors.allowCredentials | Returns as response header of OPTIONS request. | bool | false |
| grpc.interceptors.cors.exposeHeaders | Provide exposed headers returns as response header of OPTIONS request. | []string | "" |
| grpc.interceptors.cors.maxAge | Provide max age returns as response header of OPTIONS request. | int | 0 |

#### JWT
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.jwt.enabled | Enable JWT interceptor | boolean | false |
| grpc.interceptors.jwt.signingKey | Required, Provide signing key. | string | "" |
| grpc.interceptors.jwt.ignorePrefix | Provide ignoring path prefix. | []string | [] |
| grpc.interceptors.jwt.signingKeys | Provide signing keys as scheme of <key>:<value>. | []string | [] |
| grpc.interceptors.jwt.signingAlgo | Provide signing algorithm. | string | HS256 |
| grpc.interceptors.jwt.tokenLookup | Provide token lookup scheme, please see bellow description. | string | "header:Authorization" |
| grpc.interceptors.jwt.authScheme | Provide auth scheme. | string | Bearer |

The supported scheme of **tokenLookup** 

```
// Optional. Default value "header:Authorization".
// Possible values:
// - "header:<name>"
// Multiply sources example:
// - "header: Authorization,cookie: myowncookie"
```

#### Secure
Interceptor for grpc-gateway.

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.secure.enabled | Enable secure interceptor | boolean | false |
| grpc.interceptors.secure.xssProtection | X-XSS-Protection header value. | string | "1; mode=block" |
| grpc.interceptors.secure.contentTypeNosniff | X-Content-Type-Options header value. | string | nosniff |
| grpc.interceptors.secure.xFrameOptions | X-Frame-Options header value. | string | SAMEORIGIN |
| grpc.interceptors.secure.hstsMaxAge | Strict-Transport-Security header value. | int | 0 |
| grpc.interceptors.secure.hstsExcludeSubdomains | Excluding subdomains of HSTS. | bool | false |
| grpc.interceptors.secure.hstsPreloadEnabled | Enabling HSTS preload. | bool | false |
| grpc.interceptors.secure.contentSecurityPolicy | Content-Security-Policy header value. | string | "" |
| grpc.interceptors.secure.cspReportOnly | Content-Security-Policy-Report-Only header value. | bool | false |
| grpc.interceptors.secure.referrerPolicy | Referrer-Policy header value. | string | "" |
| grpc.interceptors.secure.ignorePrefix | Ignoring path prefix. | []string | [] |

#### CSRF
Interceptor for grpc-gateway.

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.csrf.enabled | Enable csrf interceptor | boolean | false |
| grpc.interceptors.csrf.tokenLength | Provide the length of the generated token. | int | 32 |
| grpc.interceptors.csrf.tokenLookup | Provide csrf token lookup rules, please see code comments for details. | string | "header:X-CSRF-Token" |
| grpc.interceptors.csrf.cookieName | Provide name of the CSRF cookie. This cookie will store CSRF token. | string | _csrf |
| grpc.interceptors.csrf.cookieDomain | Domain of the CSRF cookie. | string | "" |
| grpc.interceptors.csrf.cookiePath | Path of the CSRF cookie. | string | "" |
| grpc.interceptors.csrf.cookieMaxAge | Provide max age (in seconds) of the CSRF cookie. | int | 86400 |
| grpc.interceptors.csrf.cookieHttpOnly | Indicates if CSRF cookie is HTTP only. | bool | false |
| grpc.interceptors.csrf.cookieSameSite | Indicates SameSite mode of the CSRF cookie. Options: lax, strict, none, default | string | default |
| grpc.interceptors.csrf.ignorePrefix | Ignoring path prefix. | []string | [] |

### Development Status: Stable

### Appendix
Use bellow command to rebuild proto files, we are using [buf](https://docs.buf.build/generate-usage) to generate proto related files.
Configuration could be found at root path of project.

- make buf

### Contributing
We encourage and support an active, healthy community of contributors &mdash;
including you! Details are in the [contribution guide](CONTRIBUTING.md) and
the [code of conduct](CODE_OF_CONDUCT.md). The rk maintainers keep an eye on
issues and pull requests, but you can also report any negative conduct to
dongxuny@gmail.com. That email list is a private, safe space; even the zap
maintainers don't have access, so don't hesitate to hold us to a high
standard.

<hr>

Released under the [MIT License](LICENSE).

