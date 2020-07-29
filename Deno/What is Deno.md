<!--[TOC]: # "## Table of Contents"-->

## Table of Contents
- [기반 기술](#기반-기술)
- [Node.js 설계의 단점](#nodejs-설계의-단점)
- [Deno의 특징](#deno의-특징)
  - [ES Module만 사용](#es-module만-사용)
  - [Enhanced Security](#enhanced-security)
  - [Built-in TypeScript](#built-in-typescript)
  - [Top level await in supported](#top-level-await-in-supported)
  - [. Browser compatible](#-browser-compatible)
  - [NodeJS vs Deno](#nodejs-vs-deno)
- [Sample Code](#sample-code)

### 기반 기술
1. V8 Javascript Runtime
2. Rust (replace C++)
3. Tokio (event loop)
4. TypeScript

### Node.js 설계의 단점
NodeJS와 데노를 모두 설계한 **ryan dahl**에 따르면, NodeJS에는 다음과 같은 3가지 중요한 설계 문제가 있다고 언급합니다.
- 중앙 배포식의 잘못 설계된 모듈 시스템(NPM)
- 지원해야 하는 상당수의 레거시 API
- 보안의 결핍 (퍼미션 없이 리소스 읽고 쓰기가 가능함)

## Deno의 특징
### ES Module만 사용
기존 NodeJS에서는 ES 비표준적인 CommonJS 방식인 `require('module')` 형식으로 모듈을 로드하였지만 Deno에서는 다음과 같은 방식으로 모듈을 로드합니다.
``` typescript
import { server } from "http://deno.land/std/http/server.ts"
```

 모듈관리 부분에서 NodeJS는 NPM이라는 매니저를 별도로 두어 모듈을 관리해야하는 큰 단점이 있었습니다. 하지만 Deno의 경우는 별도의 패키지 매니저를 두지 않고 Deno 자체가 패키지 매니저 역할을 하며 서드파티 모듈을 위한 중앙 리포지토리가 없습니다.
 또한 모듈은 항상 로컬에서 캐시 되고 컴파일되며 명시적으로 갱신을 요청하지 않는 한 업데이트 되지 않습니다.
  NodeJS 처럼 Npm서버가 장애가 발생한 상황이나 인터넷이 되지 않는 환경에서도 Deno는 캐싱되어있는 모듈이 있으면 바로 사용이 가능합니다.

> deno가 실행될때 import {serve} from "https://{module path}"가 캐쉬에 없으면 최초 한번 다운로드 받는 형식
> 다운로드 받은 모듈은 다음과 같은 경로에 캐싱이 됩니다.
> - windows : `%LOCALAPPDATA%/deno`
> - mac : `$HOME/Library/Caches/deno`
> 	if something fails, it falls back to $HOME/.deno

### Enhanced Security
Deno의 실행환경(Sandbox) 이외의 외부 자원에 접근시에는 반드시 시작시에 퍼미션을 명시해주어야 합니다.
실행시에 제한된 권한만 부여하여 의도하지 않는 일이 일어나지 않도록 지정합니다.
- `--allow-read` : Allow file system read access
- `--allow-write` : Allow file system write access
- `--allow-net` : Allow network access
- `--allow-env` : Allow enviroment access
- `--allow-run` : Allow running subprocesses
- `-A`, `--allow-all` : Allow all permissions
- `--allow-read=/tmp` 또는 `--allow-net=google.com` 와 같은 세부적인 설정도 가능

NodeJS의 경우 포착되지 않는 오류 발생 하더라도 계속 허용을 해줌으로 버그가 발생한 포인트를 못찾는 어려움이 있지만 Deno의 경우 포착되지 않는 오류 발생시에도 항상 프로세스를 중지합니다.

### Built-in TypeScript
복잡한 타입스크립트 개발 환경을 만들 필요없이 기본적인 환경을 제공하여줍니다. 

### Top level await in supported
탑레벨의 코드에서 async로 감싸주는것 없이 바로 await 처리가 가능합니다.
``` typescript
import { serve } from "https://deno.land/std@0.50.0/http/server.ts";
const s = serve({ port : 8000 });
console.log("http://loalhost:8000/");
// 탑레벨에서 없이 바로 await이 가능
for await (const req of s) {
    req.respond({ body: "Hello World"})
}

// 탑레벨에서 사용하는 기존의 코드 (async로 감싸줘야함)
(async () => {
	for await (const req of s) {
		req.respond({ body : "Hello World\n" });
	}
})();
```
###. Browser compatible
Deno는 브라우저 호환성을 중시합니다.
예를들면 NodeJS에서 fetch를 써야하는 상황에서는 NPM에서 의존성을 추가해야 사용이 가능하지만 일반 브라우저 환경에서는 fetch가 내장되어있습니다.
Deno는 브라우저 호환성을 위해 Api 환경이 브라우저와 동일하게 구성이 되어있습니다.
그리고 상단에 설명한것과 같이 모듈 사용시 `require`를 사용하지 않고 ES 표준 형태인 `import` / `export` 를 사용합니다.

> 아직은 브라우저와 완벽하게 호환이 되지 않지만 지속적인 업데이트로 적용한다고 발표하였습니다.

### NodeJS vs Deno
||NodeJS|Deno|
|:-|:-|:-|
|**Engine**|V8|V8|
|**Written in**|C++, Javascript|Rust & Typescript|
|**Package managing**|npm|uses URLs|
|**importing package**|CommonJS syntax|EX Modules|
|**Security**|full access|permissioned access|
|**TypeScript support**|not build in| built in|


## Sample Code

간단하게 get, post, put 메서드를 이용하여 CRU를 구성한 코드입니다.
미들웨어로 oak를 사용하였으며 NodeJS의 Express와 굉장히 비슷합니다

``` typescript
// server.ts
import { Application, Router } from "https://deno.land/x/oak/mod.ts";
import router from "./routes.ts"

const app = new Application();

app.use(router.routes());
app.use(router.allowedMethods());

console.log(`Server is listening on port 5000`);
await app.listen({ port : 5000 });
```

임시적으로 메모리에 배열로 저장하는 코드를 만들어 요청이 올때마다 내용을 전달해줍니다.
``` typescript
// router.ts
import { Router } from "https://deno.land/x/oak/mod.ts";
import { v4 } from "https://deno.land/std/uuid/mod.ts";
import { Book } from "./types.ts"

const books: Book[] = [
    {
        id : v4.generate(),
        title : "Book One",
        author : "One"
    },
    {
        id : v4.generate(),
        title : "Book Two",
        author : "Two"
    },
    {
        id : v4.generate(),
        title : "Book Three",
        author : "Three"
    }
]

const router = new Router();
// context 내부에 req, res 둘다 들어있음
// 디스턱쳐링으로 {request, response} 형식으로 꺼낼 수 있음 
router
    .get('/', (context) => {
        context.response.body = "Hello World";
    })
    .get("/books", (context) => {
        context.response.body = books;
    })
    .post("/book", async (context) => {
        // Promiose로 반납됨
        const body = context.request.body();

        if (!context.request.hasBody) {
            context.response.status = 400;
            context.response.body = "데이터가 없습니다.";
        } else {
            const book: Book = await body.value;
			// v4는 UUID를 생성하여준다.
            book.id = v4.generate();
            context.response.status = 201
            context.response.body = book;
        }
    })
    .get("/book/:id", async (context) => {
		// :id 형태로 전달하면 context.params내부에 파싱하여 값을 넣어준다.
        const book: Book | undefined = books.find(
          (b) => b.id === context.params.id);
        if (book) {
            context.response.status = 200;
            context.response.body = book;    
        } else {
            context.response.status = 404;
            context.response.body = "책을 찾지 못했습니다.";
        }
    });

export default router
```
해당언어는 타입스크립트이므로 객체의 타입을 명시하여줍니다.
```typescript
// book.ts
export interface Book {
    id: string;
    title: string;
    author: string;
}
```
- `routers.ts` : path, methos별 요청에 대한 분기. controller 역할을 수행
- `server.ts` : server application을 구성한다.
- `types.ts` : typescript 타입을 명시

해당코드의 실행 방법은 다음과 같습니다.
``` bash
deno run --allow-net server.ts
```


> 참고 : 
- https://www.youtube.com/watch?v=BVScm6JeDk0
- http://www.itworld.co.kr/t/61023/%EA%B0%9C%EB%B0%9C%EC%9E%90/145808