---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
---

# NestJS 入門

---

# 今日のひとこと

最近読んだ漫画

- チェンソーマン（公安編まで）
- 呪術廻戦（単行本20巻まで）
- さくらの唄（下巻途中まで）

---

# Agenda

- NestJSとは
- NestJSの構成要素
- 動作・ドキュメントを見てみよう（時間あれば）
- まとめ

---

# NestJSとは

- Node.js サーバーサイドアプリケーションを構築するためのフレームワーク
- Express と Fastify のどちらかを HTTP サーバーフレームワークとして使用
- TypeScript サポート

---

# NestJSの哲学

Node.js を取り巻く現状

- Web の領域では Node.js のおかげで JS がフロントエンドとバックエンド両方で「共通語」となっている
- 特にフロントエンドでは React, Vue, Angular などで恩恵が得られた
  - 生産性の向上
  - 高速なテスト
  - 拡張性の高さ

---

# NestJSの哲学

バックエンドでは

- アーキテクチャという主要な問題を効果的に解決できていない
- NestJS で、そこら辺の問題を解決できるようにした
  - 高度にテスト可能
  - スケーラブル
  - 疎結合
  - 簡単に保守可能
- Angular に影響を受けている

---

# セットアップ方法

```
npm i -g @nestjs/cli
nest new project-name
```

自動で必要なファイルが生成される

```
├── src
│   ├── app.controller.spec.ts
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   └──  main.ts
```
---

# NestJSの構成要素

NestJS を構成する基本的な概念について
- Controllers
- Providers
- Modules
- Exception filters
- Pipes
- Validation

---

# Controllers

<img src="/Controllers.png" style="width: 600px">

- 送られてくるリクエストを処理し、レスポンスをクライアントに返す部分
- アプリケーションの特定のリクエストを受け取ることが目的
- ルーティング機能で、リクエストを各コントローラーに振り分け

---

# Controllers

GET の例

```ts
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

```ts
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get(':id')
  findOne(@Param() params): string {
    console.log(params.id);
    return `This action returns a #${params.id} cat`;
  }
}
```

---

# Controllers

POST の例

```ts
import { Controller, Post } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    return 'This action adds a new cat';
  }
}
```

クライアントパラメーターを受け取るには、 `@Body()` デコレーターを追加する

---

# Controllers

DTO(Data Transfer Object) を定義してデータをやり取りする

```ts
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}

// こういうこともできる
export class UpdateCatDto extends PartialType<CreateCatDto> {}
```

- ネットワーク上でどうデータを送信するかを定義するオブジェクト
- TypeScript の `interface` か単純なクラスで表現できる
  - `interface` だとトランスパイル時に削除されるので、Nest実行時に参照できない
  - クラスだとコンパイルされたJSの中で実体が保たれる
  - Pipes などの機能を使う際に不便になるのでクラス推奨

---

# Providers

- 依存関係として注入できるもの
- `@Injectable()` デコレーターを使用する
- 主に Services がこれにあたる
- 依存性の注入に関しては、 https://angular.io/guide/dependency-injection

---

# Providers

Services でデータの保存と取得をする例

```ts
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```

---

# Providers

Controllers へ依存関係の注入

`private` 構文で、 Service の宣言と初期化を同時にできる

```ts
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

---

# Modules

<img src="/Modules.png" style="width: 550px">

- `@Module()` デコレーターでアノテーションされたクラス
- Nest がアプリケーションの構造を整理するためのメタデータを提供する

---

# Modules

同じドメインで束ねてコードを整理していく

```ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

```ts
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```

---

# Exception filters

アプリケーション全体で処理されない全ての例外を処理できる

<img src="/ExceptionFilters.png" style="width: 550px">

---

# Exception filters

例外発生時に、自動的にレスポンスを送信できる

```json
{
  "statusCode": 500,
  "message": "Internal server error"
}
```

---

# Exception filters

標準的な例外処理

- `HttpException` クラスが用意されている
  - response: `message` に入ってくる文字列 or オブジェクトを指定する
  - status: HTTP のステータスコードを定義する

---

# Exception filters

標準的な例外処理

```ts
@Get()
async findAll() {
  throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);
}
```

```json
{
  "statusCode": 403,
  "message": "Forbidden"
}
```

---

# Exception filters

カスタム例外処理

`HttpException` クラスを継承することでカスタム例外処理も定義できる

```ts
export class ForbiddenException extends HttpException {
  constructor() {
    super('Forbidden', HttpStatus.FORBIDDEN);
  }
}
```

```ts
@Get()
async findAll() {
  throw new ForbiddenException();
}
```

---

# Pipes

<img src="/Pipes.png" style="width: 550px">

- 変換(Transform)
  - 入力データを目的の形式に変換する（例: 文字列から整数値へ）
- 検証(Validation)

---

# Pipes

Nest にはいくつかの組み込みパイプが用意されている

- ValidationPipe
- ParseIntPipe
- ParseFloatPipe
- ParseBoolPipe
- ParseArrayPipe
- ParseUUIDPipe
- ParseEnumPipe
- DefaultValuePipe
- ParseFilePipe

---

# Pipes

使用方法

適切なコンテキストにバインド、パイプを特定のルートハンドラーメソッドに関連付け、<br>
そのメソッドが呼ばれる前に実行されるようにする

```ts
@Get(':id')
async findOne(@Param('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```

---

# Pipes

仮にパラメーターに文字列が渡ってきた場合

```
GET localhost:3000/abc
```

```json
{
  "statusCode": 400,
  "message": "Validation failed (numeric string is expected)",
  "error": "Bad Request"
}
```

---

# Validation

組み込みの `ValidationPipe` を使うとよりきめ細やかな Validation ができる

```shell
npm i --save class-validator class-transformer
```

`main.ts` で `ValidationPipe` をバインドする

```ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(3000);
}
bootstrap();
```

---

# Validation

使用方法

```ts
import { IsEmail, IsNotEmpty } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsNotEmpty()
  password: string;
}
```

Email として不正な文字列が渡った場合

```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": ["email must be an email"]
}
```

---

# Validation

使用方法

```ts
import { IsNumberString } from 'class-validator';

export class FindOneParams {
  @IsNumberString()
  id: number;
}
```

渡ってくるパラメーターにも使用可能

```ts
@Get(':id')
findOne(@Param() params: FindOneParams) {
  return 'This action returns a user';
}
```

---

# Validation

使用方法

オプションも用意されている<br>
https://docs.nestjs.com/techniques/validation#using-the-built-in-validationpipe

```ts
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    forbidNonWhiteListed: true,
  }),
);
```

---

# ちょっと動作を見てみよう

---

# Documentちょっと目を通してみよう

---

# まとめ

- 形式が決まってるので誰が書いても変わらない
- 例外処理も組み込みを駆使して簡単に可能
- ドキュメントが充実!