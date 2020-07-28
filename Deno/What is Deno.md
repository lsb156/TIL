# Deno

<!--[TOC]: # "## Table of Contents"-->

## Table of Contents
- [Deno란?](#deno란)
  - [기반 기술](#기반-기술)
  - [Node.js 설계의 단점](#nodejs-설계의-단점)
- [Deno의 특징](#deno의-특징)
  - [ES Module만 사용](#es-module만-사용)
  - [Enhanced Security](#enhanced-security)
  - [Built-in TypeScript](#built-in-typescript)
  - [Top level await in supported](#top-level-await-in-supported)
  - [. Browser compatible](#-browser-compatible)
  - [NodeJS vs Deno](#nodejs-vs-deno)


## Deno란?
### 기반 기술
1. V8 Javascript Runtime
2. Rust (replace C++)
3. Tokio (event loop)
4. TypeScript

### Node.js 설계의 단점
Node.js와 데노를 모두 설계한 달에 따르면, Node.js에는 다음과 같은 3가지 중요한 설계 문제가 있다.
- 중앙 배포식의 잘못 설계된 모듈 시스템
- 지원해야 하는 상당수의 레거시 API
- 보안의 결핍 (퍼미션 없이 리소스 읽고 쓰기가 가능함)

## Deno의 특징
### ES Module만 사용
기존 Nodejs에서는 ES 비표준적인 CommonJS 방식인 `require('module')` 형식으로 모듈을 로드하였지만 Deno에서는 다음과 같은 방식으로 모듈을 로드한다.
``` typescript
import { server } from "http://deno.land/std/http/server.ts"
```

 모듈관리 부분에서 Nodejs는 NPM이라는 매니저를 별도로 두어 모듈을 관리해야하는 큰 단점이 있었다. 하지만 Deno의 경우는 별도의 패키지 매니저를 두지 않고 Deno 자체가 패키지 매니저 역할을 하며 서드파티 모듈을 위한 중앙 리포지토리가 없다.
 또한 모듈은 항상 로컬에서 캐시 되고 컴파일되며 명시적으로 갱신을 요청하지 않는 한 업데이트 되지 않는다.
  Nodejs 처럼 Npm서버가 장애가 발생한 상황이나 인터넷이 되지 않는 환경에서도 Deno는 캐싱되어있는 모듈이 있으면 바로 사용이 가능하다.

> deno가 실행될때 import {serve} from "https://{module path}"가 캐쉬에 없으면 최초 한번 다운로드 받는다.
> 다운로드 받은 모듈은 다음과 같은 경로에 캐싱이된다.
> - windows : `%LOCALAPPDATA%/deno`
> - mac : `$HOME/Library/Caches/deno`
> 	if something fails, it falls back to $HOME/.deno

### Enhanced Security
Deno의 실행환경(Sandbox) 이외의 외부 자원에 접근시에는 반드시 퍼미션을 따로 받아야한다.
- `--allow-read` : Allow file system read access
- `--allow-write` : Allow file system write access
- `--allow-net` : Allow network access
- `--allow-env` : Allow enviroment access
- `--allow-run` : Allow running subprocesses
- `-A`, `--allow-all` : Allow all permissions
- `--allow-read=/tmp` 또는 `--allow-net=google.com` 와 같은 세부적인 설정도 가능

Nodejs의 경우 포착되지 않는 오류 발생 시 실행을 계속 허용을 해줌으로 버그를 차즌ㄴ 포인트를 못찾는 어려움이 있지만 Deno의 경우 포착되지 않는 오류 발생시에도 항상 프로세스를 중지한다.

### Built-in TypeScript
복잡한 타입스크립트 개발 환경을 만들 필요없이 기본적인 환경을 제공하여준다.

### Top level await in supported
탑레벨의 코드에서 async로 감싸주는것 없이 바로 await 처리가 가능하다.
``` typescript
import { serve } from "https://deno.land/std@0.50.0/http/server.ts";
const s = serve({ port : 8000 })l
console.log("http://loalhost:8000/");
for await (const req of s) {
    req.respond({ body: "Hello World"})
}
```
	- 탑레벨에서 사용하는 기존의 코드 (async로 감싸줘야함)
``` typescript
(async () => {
	for await (const req of s) {
		req.respond({ body : "Hello World\n" });
	}
})();
```
###. Browser compatible
Deno는 브라우저 호환성을 중시하고있다.
예를들면 NodeJs에서 fetch를 써야하는 상황에서는 npm에서 의존성을 추가해야 사용이 가능하지만 일반 브라우저 환경에서는 fetch가 내장되어있다.
Deno는 브라우저 호환성을 위해 Api 환경이 동일하게 구성이 되었다.
그리고 위에서 설명한것과 같이 require를 사용하지 않고 import / export 를 사용함으로써 ES 표준에 맞게 사용이 된다.

> 아직은 브라우저와 완벽하게 호환이 되지 않지만 지속적인 업뎃으로 완성해나가고 있다 - v1.0.0

### NodeJS vs Deno
|title|Node|Deno|
|:-|:-|:-|
|Engine|V8|V8|
|Written in|C++, Javascript|Rust & Typescript|
|Package managing|npm|uses URLs|
|importing package|CommonJS syntax|EX Modules|
|Security|full access|permissioned access|
|TypeScript support|not build in| built in|


