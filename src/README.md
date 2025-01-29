# Vite

-   다른 정적 파일 서버와 크게 다르지 않지만, 기존 번들러에서 제공하는 기능을 그대로 제공

## npm을 이용한 디펜던시 임포트와 사전 번들링

```js
import { someMethod } from 'my-dep';
```

네이티브 JavaScript에서는 위의 코드는 모듈의 위치를 찾을 수 없기 때문에 정상적으로 실행되지 않는다.

하지만, vite의 경우 아래의 기준을 통해서 가져오기 때문에 위의 코드가 정상적으로 실행된다.

1. 사전 번들링(디펜던시 코드)을 통해 페이지 로딩 속도를 개선하고, CommonJS/UMD 모듈을 ESM으로 변환한다. --> JS 기반의 다른 번들러보다 빠른 콜드 스타드가 가능

2. `/node_modules/.vite/deps/my-dep.js?v=f3sf2ebd`와 같이 URL을 이용해 ESM을 지원하는 브라우저에서 모듈을 가져올 수 있도록 import 구문을 수정

```
※. 디펜던시는 반드시 캐시된다.

HTTP 헤더를 이용해 요청한 디펜던시를 브라우저에서 캐싱하도록 한다

디펜던시의 수정 및 디버깅이 필요할 경우에는 --force 옵션을 이용해 개발 서버를 재시작한다.

npm run dev --force

```

## Hot Module Replacement

ESM을 통해 HMR API를 제공, 일반적인 사용자의 경우 설정에 대해 크게 신경쓰지 않아도 된다.

## TypeScript

1. 트랜스파일만 수행

`.ts`에 대해서 트랜스파일링만 수행, 타입 검사는 IDE와 빌드시에 수행된다.

타입 검사를 하지 않는 이유는 파일 단위로 작동할 수 있기 떄문인 반면, 타입 검사는 전체 모듈 그래프에 대한 탐색이 필요하다. Vite의 변환 과정에 타입 검사를 추가하게 된다면, Vite의 장점인 속도가 사라지게 될 것.

만일, 타입 검사를 하고 싶을 경우에는 다음의 옵션이 있다.

-   `tsc --noEmit` : Vite 빌드 명령어에 추가하여 타입 검사만하고, 빌드 결과물을 생성하지 않는다.

-   `tsc --noEmit --watch` : 파일이 변경될 때마다 자동으로 타입 검사를 실행 --> 개발 서버 실행시, 별도로 타입 검사를 돌리고 싶을 경우 유용

-   `vite-plugin-checker` : 플러그인, 타입 에러를 브라우저 콘솔에서 확인

<br />

추가로, 타입만을 가져오는 경우에는 잘못 번들링 될 수 있는데, 타입 전용 Imports와 Exports를 사용하면 이 문제를 우회할 수 있다.

```js
import type { T } from 'only/types';
export type { T };
```

## 타입스크립트 컴파일러 옵션

**1. isolateModules**

```typescript
const enum Colors {
    Red: "RED",
    Blue: "BLUE"
}

console.log(Colors.Red);
```

esbuild는 타입 분석을 하지 않기 때문에, 위의 코드를 JS로 변환할 경우 에러가 발생한다.

이를 경고하지 위해서, `tsconfig.json`의 compilerOptions 설정에 `"isolatedModules": true`와 같이 설정해줘야하며, TS가 위의 상황에서 작동하지 않는 경우를 경고할 수 있다.

일부 라이브러리의 경우 위의 설정으로 타입 체크가 정상적으로 동작하지 않을 수 있다. 이 때는 `"skipLibCheck": true`를 사용해 오류가 발생되지 않도록 한다.

<br />

**2. useDefineForClassFields**

추후 학습예정

## 정적 에셋

정적 에셋을 import 하는 경우, Public URL이 반환

```js
import imgUrl from './img.png';

// . = public 경로를 가리킴
```

URL 쿼리를 이용해 에셋을 가져올 때 어떻게 가져올 것인지 명시 가능

```js
// URL로 에셋 가져오기
import assetAsURL from './asset.js?url';

// String 타입으로 에셋 가져오기
import assetAsURL from './asset.js?raw';

// 웹 워커 가져오기
import assetAsURL from './asset.js?worker';

// Base64 포맷의 문자열 형태로 웹 워커 가져오기
import assetAsURL from './asset.js?worker&inline';
```

## JSON

JSON 파일은 바로 import가 가능, 가져올 필드를 지정할 수 있음

```js
import json from './example.json';

import { example, user } from './example.json';
```

## Glob Import

`import.meta.glob` 함수를 이용해 여러 모듈을 한 번에 가져올 수 잇도록 지원

```js
const modules = import.meta.glob('./dir/*.js');

// vite를 통해 변환된 코드
const modules = {
    './dir/foo.js': () => import('./dir/foo.js'),
    './dir/bar.js': () => import('./dir/bar.js'),
};

// modules를 순회하여 각 모듈에 접근할 수 있음
for (const path in modules) {
    modules[path]().then((mod) => {
        console.log(path, mod);
    });
}
```

동적이 아닌 직접 모듈을 가져오고 싶다면, 두 번째 인자로 { eager: true } 객체를 전달

```js
const modules = import.meta.glob('./dir/*.js', { eager: true });

// Vite를 통해 변환된 코드
import * as __glob__0_0 from './dir/foo.js';
import * as __glob__0_1 from './dir/bar.js';
const modules = {
    './dir/foo.js': __glob__0_0,
    './dir/bar.js': __glob__0_1,
};
```

위의 기능들은 vite에서 제공하는 기능이며, ES 표준에서 지원하지 않는다.

추가적인 내용은 공식문서를 참조

## 동적 Import

Glob Import와 마찬가지로 Vite는 변수를 사용한 동적인 Import도 지원한다.

```js
const module = await import(`./dir/${file}.js`);
```
