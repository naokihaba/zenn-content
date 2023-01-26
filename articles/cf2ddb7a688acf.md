---
title: "【NestJS】Responseデータの形式をInterceptorでいい感じに変換する方法"
emoji: "🗂️"
type: "tech"
topics:
  - "typescript"
  - "nestjs"
  - "restapi"
published: true
published_at: "2022-03-22 03:50"
---

# 概要

`APIエンドポイント`から返却されるレスポンスデータの形式を`Interceptor`で変換した

```json
{
  "message": "success",
  "statusCode": 200,
  "errors": ""
}
```

https://docs.nestjs.com/interceptors

# `Interceptor`を定義する

```ts:transform.interceptor.ts
import {Injectable, NestInterceptor, ExecutionContext, CallHandler} from '@nestjs/common';
import {Observable} from 'rxjs';
import {map} from 'rxjs/operators';

// オブジェクトのexport
export const Info = {
    statusCode: 200,
    message: 'success'
}

/**
 * export type 型エイリアスのexport
 * オブジェクトの値のユニオン型を作成
 * typeof xxx &  型定義をマージして返却
 */
export type Response<T> = typeof Info & {
    data: T
}

/**
 * Object.assign で空オブジェクトに指定した形式で代入
 */
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
    intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
        return next.handle().pipe(map(data => Object.assign({}, Info, {data})));
    }
}
```

# 定義した`Interceptor`を`main.ts`で読み込む

> コア関数NestFactoryを使用してNestアプリケーションのインスタンスを生成するアプリケーションのエントリファイルです。

The entry file of the application which uses the core function NestFactory to create a Nest application instance.

https://docs.nestjs.com/first-steps

```ts:main.ts
async function bootstrap() {
    const app = await NestFactory.create(AppModule, {
        logger: loggerFactory.useFactory('warn'),
    })
    app.useGlobalInterceptors(new TransformInterceptor())
}
```
