# nestjs

## 介绍

Nest（NestJS）是一个框架，用于构建高效，可扩展的 Node.js 服务器端应用程序。它使用渐进式 JavaScript，使用 TypeScript 构建并完全支持 TypeScript（但仍使开发人员能够使用纯 JavaScript 编码），并结合了 OOP（面向对象编程），FP（函数式编程）和 FRP（函数式响应式编程）的元素。

在引擎盖下，Nest 使用强大的 HTTP 服务器框架，如 Express（默认），并且可以选择配置为使用 Fastify！

Nest 提供了高于这些常见 Node.js 框架（Express/Fastify）的抽象级别，但也直接向开发人员公开其 API。这使开发人员可以自由使用可用于底层平台的大量第三方模块。<br/>

[NestJs 官方文档](https://docs.nestjs.com/)<br/>
[NestJs 国内文档](https://docs.nestjs.cn)<hr/>

## 安装

node 版本>12

```npm
npm i -g @nestjs/cli
nest new project-name
```

### nestjs/cli 命令

```cmd
nest g resource [name] // crud模块生成器
nest g middleware [name] //生成中间件
nest g guard [name] // 生成守卫
```
## nest 数据传输方式

对于前端来说，后端主要是提供 http 接口来传输数据，而这种数据传输的方式主要有 5 种：

1. url param
2. query
3. form-urlencoded
4. form-data
5. json

### url param

```javascript
http://guang.zxg/person/1111   // 参数为111
```

nest 解析

```typescript
@Controller('api/person')
export class PersonController {
  @Get(':id')
  urlParam(@Param('id') id: string) {
    return `received: id=${id}`
  }
}
```

@Controller('api/person') 的路由和 @Get(':id') 的路由会拼到一起，也就是只有 /api/person/xxx 的 get 请求才会走到这个方法。

前端代码

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <script src="https://unpkg.com/axios@0.24.0/dist/axios.min.js"></script>
  </head>
  <body>
    <script>
      async function urlParam() {
        const res = await axios.get('/api/person/1')
        console.log(res)
      }
      urlParam()
    </script>
  </body>
</html>
```

### query

通过 url 中 ？后面的用 & 分隔的字符串传递数据。比如：

```
http://guang.zxg/person?name=guang&age=20
```

这里的 name 和 age 就是 query 传递的数据。

nest 解析

```typescript
@Controller('api/person')
export class PersonController {
  @Get('find')
  query(@Query('name') name: string, @Query('age') age: number) {
    return `received: name=${name},age=${age}`
  }
}
```

前端代码

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <script src="https://unpkg.com/axios@0.24.0/dist/axios.min.js"></script>
  </head>
  <body>
    <script>
      async function query() {
        const res = await axios.get('/api/person/find', {
          params: {
            name: 'ayu',
            age: 20,
          },
        })
        console.log(res)
      }
      query()
    </script>
  </body>
</html>
```

### form urlencoded

form urlencoded 是通过 body 传输数据，其实是把 query 字符串放在了 body 里，所以需要做 url encode：

前端代码

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <script src="https://unpkg.com/axios@0.24.0/dist/axios.min.js"></script>
    <script src="https://unpkg.com/qs@6.10.2/dist/qs.js"></script>
  </head>
  <body>
    <script>
      async function formUrlEncoded() {
        const res = await axios.post(
          '/api/person',
          Qs.stringify({
            name: '光',
            age: 20,
          }),
          {
            headers: { 'content-type': 'application/x-www-form-urlencoded' },
          }
        )
        console.log(res)
      }

      formUrlEncoded()
    </script>
  </body>
</html>
```

nest 解析

```typescript
@Controller('api/person')
export class PersonController {
  @Post()
  body(@Body() createPersonDto) {
    return `received: ${JSON.stringify(createPersonDto)}`
  }
}
```

### json

nest 解析

```typescript
@Controller('api/person')
export class PersonController {
  @Post()
  body(@Body() createPersonDto) {
    return `received: ${JSON.stringify(createPersonDto)}`
  }
}
```

前端代码使用 axios 发送 post 请求，默认传输 json 就会指定 content type 为 application/json，不需要手动指定：

前端代码

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <script src="https://unpkg.com/axios@0.24.0/dist/axios.min.js"></script>
  </head>
  <body>
    <script>
      async function json() {
        const res = await axios.post('/api/person', {
          name: 'ayu',
          age: 20,
        })
        console.log(res)
      }
      json()
    </script>
  </body>
</html>
```

**总结**
我们用 axios 发送请求，使用 Nest 起后端服务，实现了 4 种 http/https 的数据传输方式：

其中前两种是 url 中的：

url param： url 中的参数，Nest 中使用 @Param 来取
query：url 中 ? 后的字符串，Nest 中使用 @Query 来取
后三种是 body 中的：

form urlencoded： 类似 query 字符串，只不过是放在 body 中。Nest 中使用 @Body 来取，axios 中需要指定 content type 为 application/x-www-form-urlencoded，并且对数据用 qs 或者 query-string 库做 url encode
json： json 格式的数据。Nest 中使用 @Body 来取，axios 中不需要单独指定 content type，axios 内部会处理。

## provider

Nest 实现了 IOC 容器，会从入口模块开始扫描，分析 Module 之间的引用关系，对象之间的依赖关系，自动把 provider 注入到目标对象。<br/>
provider 一般都是用 @Injectable 修饰的 class：

```typescript
import { Injectable } from '@nestjs/common'

@Injectable()
export class PersonService {}
```

在 module 里面的 provide 声明

```typescript
// app.modules.ts
import { Module } from '@nestjs/common'
import { AppController } from './app.controller'
import { AppService } from './app.service'
import { PersonModule } from './person/person.module'

@Module({
  imports: [PersonModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

此外 provide 还可以自定义名字以及自定义值

```typescript

@Module({
  controllers: [UserController],
  providers: [{
    provide: "Xiaoman", // inject参数
    useClass: UserService // 值
  }, {
    provide: "JD",
    useValue: ['TB', 'PDD', 'JD'] // 自定义值
  }]

```

使用 provide 内注入的值

```typescript
// User.controller.ts
@Controller()
export class AppController {
  constructor(
    private readonly appService: AppService
    @Inject('Xiaoman')  private readonly userService: UserService
    @Inject('Xiaoman')  private readonly JD:string[]
    ) {}
}
```

在此使用@inject 修饰器，会自动帮我们把 provide 的值设为 controller 的实例属性/方法,因此我们也可以不设置 constructor，直接使用修饰器即可，对上面代码做简单修改

```typescript
// User.controller.ts
@Controller()
export class AppController {
  @Inject(AppService) private readonly appService: AppService
  @Inject('Xiaoman') private readonly userService: UserService
  @Inject('Xiaoman') private readonly JD: string[]
}
// 这样也是完全相同的，可以正常使用注入的方法
```

**总结**
一般情况下，provider 是通过 @Injectable 声明，然后在 @Module 的 providers 数组里注册的 class。

默认的 token 就是 class，这样不用使用 @Inject 来指定注入的 token。

但也可以用字符串类型的 token，不过注入的时候要用 @Inject 单独指定。

除了可以用 useClass 指定注入的 class，还可以用 useValue 直接指定注入的对象。

如果想动态生成对象，可以使用 useFactory，它的参数也注入 IOC 容器中的对象，然后动态返回 provider 的对象。

如果想起别名，可以用 useExisting 给已有的 token，指定一个新 token。

灵活运用这些 provider 类型，就可以利用 Nest 的 IOC 容器中注入任何对象。
## Module

每个 Nest 应用程序至少有一个模块，即根模块。根模块是 Nest 开始安排应用程序树的地方。事实上，根模块可能是应用程序中唯一的模块，特别是当应用程序很小时，但是对于大型程序来说这是没有意义的。在大多数情况下，您将拥有多个模块，每个模块都有一组紧密相关的功能

### 1.基本模块

```typescript
@Module({
  imports: [PersonModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

可以看到一个模块基本由 imports controllers 以及 providers 组成

### 2.共享模板

假如 app 模块想使用他子集下面的 user 模块的方法<br/>
<br/>

第一步将 userService 暴露出去

```typescript
@Module({
  controllers: [UserController],
  providers: [UserService],
  exports: [UserService],
})
export class AppModule {}
```

由于 user 模块在创建的时候已经注入到 app 模块了，因此可以直接在 app 模块中使用 user 模块的方法，
只需要使用 inject 声明引用便可使用

```typescript
 @Inject(PersonService)
  private readonly personService: PersonService;
  //这个时候personServie已经成为app.controller类中的实例方法了
```

那如果是平级的 user 模块想要使用 person 模块呢？<br/>
由于 2 则并没有任何关联，因此需要显示将 person 模块注入进 user 模块

```typescript
// User.modules.ts
import { PersonService } from 'src/person/person.service';
@Module({
  controllers: [UserController],
  providers: [UserService, PersonService],
})
```

### 3.全局模块

假如想让一个模块变为全局模块，那只需在模块前添加 global 装饰器即可

```typescript
@global
@module({
  controller:[...]
  providers:[...]
  exports:[...] // 不能忘记哦还是需要暴露出去
})
```

### 4.动态模块

动态模块主要就是为了给模块传递参数 可以给该模块添加一个静态方法 用来接受参数

```typescript
@Global()
@Module({})
export class ConfigModule {
  static forRoot(options: Options): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: 'Config',
          useValue: { baseApi: '/api' + options.path },
        },
      ],
      exports: [
        {
          provide: 'Config',
          useValue: { baseApi: '/api' + options.path },
        },
      ],
    }
  }
}
```

然后使用 import 注入传参即可
## 中间件

中间件是在路由处理程序之前调用的函数。中间件函数可以访问请求和响应对象，以及应用程序请求-响应周期中的 next() 中间件函数。下一个中间件函数通常由名为 的 next 变量表示。<br/>

创建一个中间件

```typescript
import { Injectable, NestMiddleware } from '@nestjs/common'

import { Request, Response, NextFunction } from 'express'

@Injectable()
export class Logger implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(req)
    next()
  }
}
```

注入中间件

```typescript
//Person.module.ts
...
interface forRoutes {
  path: string;
  method: RequestMethod;
  version?: VersionValue;
}
export declare enum RequestMethod {
    GET = 0,
    POST = 1,
    PUT = 2,
    DELETE = 3,
    PATCH = 4,
    ALL = 5,
    OPTIONS = 6,
    HEAD = 7,
    SEARCH = 8
}

export class PersonModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerWare).forRoutes('person') // forRoutes:string | forRoutes
  }
}
```

forRoutes 还支持通配符

```typescript
forRoutes({ path: 'ab*cd', method: RequestMethod.ALL })
```

'ab*cd' 路由路径将匹配 abcd 、 ab_cd abecd 、 等。字符 ? 、 + 、 * 和 () 可以在路由路径中使用，并且是其正则表达式对应项的子集。连字符 （ - ） 和点 （ . ） 由基于字符串的路径逐字解释。<br/>

[Middleware 详细内容](https://docs.nestjs.com/middleware#middleware)
## 异常筛选器

Nest 带有一个内置的异常层，负责处理整个应用程序中的所有未经处理的异常。当应用程序代码未处理异常时，该层会捕获该异常，然后自动发送适当的用户友好响应。<br/>

虽然基本（内置）异常筛选器可以自动为您处理许多情况，但我们可能希望完全控制异常层。例如，您可能希望添加日志记录或根据某些动态因素使用不同的 JSON 架构。异常筛选器正是为此目的而设计的。它们允许您控制确切的控制流以及发送回客户端的响应内容。

```typescript
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
} from '@nestjs/common'
import { Request, Response } from 'express'

@Catch(HttpException)
// 异常赛选器
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp()
    const response = ctx.getResponse<Response>()
    const request = ctx.getRequest<Request>()
    const status = exception.getStatus()

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
    })
  }
}
```

### 使用方式

#### 1. 全局使用

```typescript
// main.ts  全局使用    app.useGlobalFilters(new HttpExceptionFilter())
async function bootstrap() {
  const app = await NestFactory.create(AppModule)
  app.useGlobalFilters(new HttpExceptionFilter())
  await app.listen(3000)
}
bootstrap()
```

#### 2. 局部使用

```typescript
// person.controller.ts 局部使用
@Controller('person')
@UseFilters(new HttpExceptionFilter())
export class PersonController {}
```

#### 3. 动态使用

```typescript
// person.controller.ts 动态使用
@Controller('person')
export class PersonController {
  @Get()
  @UseFilters(new HttpExceptionFilter()) // 最好使用类，这样可以有缓存 HttpExceptionFilter
  findAll() {
    throw new HttpException('Forbidden', HttpStatus.FORBIDDEN)
  }
}
```
## 管道

管道有两个典型的用例：<br/>
转换：将输入数据转换为所需的形式（例如，从字符串转换为整数）<br/>
验证：评估输入数据，如果有效，只需原封不动地传递它;否则，引发异常

### 管道验证

```typescript
@Get(':id')
async findOne(@Param('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```

假如路由调用方式如下

```typescript
GET localhost:3000/abc
```

那么会抛出异常

```typescript
{
  "statusCode": 400,
  "message": "Validation failed (numeric string is expected)",
  "error": "Bad Request"
}
```

### 管道验证-类验证器

Nest 与类验证器库配合得很好。这个强大的库允许您使用基于装饰器的验证。基于装饰器的验证非常强大，特别是当与 Nest 的管道功能结合使用时，因为我们可以访问 metatype 已处理的属性

```npm
npm i class-validator class-transformer
```

```typescript
// person.create.dto.ts
import { IsString, IsEmail, IsNotEmpty } from 'class-validator'

export class CreatePersonDto {
  @IsNotEmpty({ message: '姓名不能为空' }) // 验证是否为空
  @IsString({ message: '参数类型错误' }) // 验证是否为字符串
  name: string

  @IsString({ message: 'age参数需为数字类型' }) // 验证是否为字符串
  age: string
  @IsEmail({}, { message: '邮箱格式不正确' }) // 验证是否为字符串
  email: string
}
```

让我们先写一个内置的 Pipes

```typescript
// validation.pipe.ts
import {
  PipeTransform,
  Injectable,
  ArgumentMetadata,
  BadRequestException,
  HttpStatus,
} from '@nestjs/common'
import { validate } from 'class-validator'
import { plainToInstance } from 'class-transformer'
import { HttpException } from '@nestjs/common'

@Injectable()
export class ValidationPipe implements PipeTransform<any> {
  async transform(value: any, { metatype }: ArgumentMetadata) {
    // value:  { name: '', age: '20', email: '2382839439@qq.com' }前端传入的参数
    // metatype:  [class CreatePersonDto] // 参数的类型
    if (!metatype || !this.toValidate(metatype)) {
      return value
    }
    const object = plainToInstance(metatype, value)
    const errors = await validate(object)
    // console.log(errors);
    if (errors.length > 0) {
      // 抛出错误对象,利用异常筛选器来接收然后返回给前端错误对象
      // HttpStatus.BAD_REQUEST规定为参数错误
      throw new HttpException(errors, HttpStatus.BAD_REQUEST, {
        description: '参数异常',
      })
    }
    return value
  }

  private toValidate(metatype: Function): boolean {
    const types: Function[] = [String, Boolean, Number, Array, Object]
    return !types.includes(metatype)
  }
}
```

使用内置 pipes

```typescript
// person.controller.ts
import {ValidationPipe} from './common/validation.pipe'
 @Post()
  @HttpCode(200)
  async create(@Body(new ValidationPipe()) CreatepersonDto: CreatePersonDto) {
    // console.log(CreatepersonDto); // { name: 'ayu', age: '20', email: '2382839439@qq.com' }
    return `received: ${JSON.stringify(CreatepersonDto)}`;
  }
```

### 全局管道

当然，nestjs 也可以使用全局管道 我们只需要将 create.dto.ts 封装好即可

```typescript
// main.ts
import { ValidationPipe } from '@nestjs/common'
app.useGlobalPipes(new ValidationPipe())
```

### 管道修饰

验证并不是自定义管道的唯一用例。在本章开头，我们使用 nest 内置的 ParseIntPipe 将传入的值进行修饰，我们通过实现 ParseIntPipe 来学习如何做管道修饰

```typescript
// parse-int.pipe.ts
import {
  PipeTransform,
  Injectable,
  ArgumentMetadata,
  BadRequestException,
} from '@nestjs/common'

@Injectable()
export class MyParseIntPipe implements PipeTransform<string, number> {
  transform(value: string, metadata: ArgumentMetadata): number {
    // console.log('metadata: ', metadata); // metadata:  { type: 'param', metatype: [Function: Number], data: 'id' }
    // console.log('value: ', value); // value: 前端传入的参数
    // you can do something here before return value
    const val = parseInt(value, 10)
    if (isNaN(val)) {
      throw new BadRequestException('参数不符合规范')
    }
    return val
  }
}
```
## 拦截器

拦截器是用装饰器注释 @Injectable() 并实现接口的 NestInterceptor 类。<br/>

拦截器的功能

1. 在方法执行之前/之后绑定额外的逻辑
2. 转换从处理程序方法返回的结果
3. 转换从处理程序抛出的异常
4. 扩展基本的处理程序类以执行额外的任务
5. 根据特定条件完全覆盖函数（例如，出于缓存目的）

###规范响应返回值

```typescript
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from '@nestjs/common'
import { Observable } from 'rxjs'
import { map } from 'rxjs/operators'

export interface Response<T> {
  data: T
}

@Injectable()
export class TransformInterceptor<T>
  implements NestInterceptor<T, Response<T>>
{
  intercept(
    context: ExecutionContext,
    next: CallHandler
  ): Observable<Response<T>> {
    return next
      .handle()
      .pipe(map((data) => ({ data, status: 0, success: true, msg: '成功' })))
  }
}
```

**使用**

1. 局部使用

```typescript
// cats.controller.ts
@UseInterceptors(TransformInterceptor)
export class CatsController {}
```

2. 全局使用

```typescript
// main.ts
app.useGlobalInterceptors(new TransformInterceptor())
```
## 警卫

守卫是用 @Injectable() 装饰器注释的类，它实现 CanActivate 接口<br/>

警卫只有一个责任。它们确定给定的请求是否将由路由处理程序处理，具体取决于运行时存在的某些条件（如权限、角色、ACL 等）。这通常称为授权。授权（及其表亲身份验证，它通常与之协作）通常由传统 Express 应用程序中的中间件处理。中间件是身份验证的不错选择，因为令牌验证和将属性附加到 request 对象等内容与特定路由上下文（及其元数据）没有很强的联系

**通过一个例子来说明警卫的作用**

```typescript
// auth.guard.ts
import {
  Injectable,
  CanActivate,
  ExecutionContext,
  UnauthorizedException,
} from '@nestjs/common'
import { Reflector } from '@nestjs/core'
import { Request } from 'express'
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('role', context.getHandler())
    // 指派的守卫验证

    // roles => ['admin']
    if (!roles) {
      return true
    }
    const request: Request = context.switchToHttp().getRequest()
    // 验证是否存在token

    //request.headers.authorization  => Bearer 121218218219219219281
    const token = request.headers.authorization
    console.log(token, roles)
    return matchRoles(roles, token)
    // 如果守卫验证失败我们可以加上自定义的错误信息
    // throw new UnauthorizedException('无权限访问');
  }
}
function matchRoles(roles: string[], token: string) {
  // verify
  // do something
  return true
}
```

需要使用警卫的 controller

```typescript
// person.controller.ts
// @UseGuards(new RolesGuard(new Reflector()))
// @UseGuards(RolesGuard)
@Controller('person')
export class PersonController {
  private readonly personService: PersonService
  @Post()
  @SetMetadata('role', ['admin']) // 设置的警卫验证的角色
  async create(@Body() CreatepersonDto: CreatePersonDto) {
    // console.log(CreatepersonDto); // { name: 'ayu', age: '20', email: '2382839439@qq.com' }
    return `received: ${JSON.stringify(CreatepersonDto)}`
  }
}
```

在 person.controller.ts 中，我们使用了@SetMetadata()装饰器来设置警卫验证的角色，然后在 auth.guard.ts 中使用@Reflect()装饰器来获取这个角色，然后进行验证，如果验证成功则返回 true，否则返回 false，如果返回 false 则会抛出一个异常，我们可以在 auth.guard.ts 中加上自定义的错误信息，如上面的代码所示，如果我们不加上自定义的错误信息，那么会抛出一个默认的错误信息，如下所示：

```typescript
{
  "statusCode": 401,
  "message": "Unauthorized"
}
```

并且在 person.controller.ts 中，我们使用了@UseGuards()装饰器来使用警卫，我们可以在@UseGuards()装饰器中传入一个或多个警卫，如上面的代码所示，我们传入了一个 RolesGuard 警卫，这样就可以对 person.controller.ts 中的 create()方法进行验证了，可以通过直接写类的形式，但是如果使用实例的形式就必须传入参数 new Reflector()

**全局警卫**
我们不仅可以局部对 controller 进行警卫设置，还可以设置全局通用警卫

```typescript
// main.ts
import { RolesGuard } from './common/auth.guard';
import { Reflector } from '@nestjs/core';
...
app.useGlobalGuards(new RolesGuard(new Reflector()));//我使用的时候必须要携带这个参数
```
## 自定义装饰器

经过了前面的学习，我们发现 nestjs 的装饰器是非常强大的，但是有时候我们需要自定义装饰器，来实现一些特殊的功能<br/>

我们先简单实现一个装饰器

```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common'
import { Request } from 'express'
// 自定义装饰器
export const User = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    // data是传入的参数，ctx是上下文
    const request: Request = ctx.switchToHttp().getRequest() // 请求对象
    // 我们可以通过传入的data来当返回的对象的key，然后返回特定的值
    return request.originalUrl // get something to return
  }
)
```

然后就可以直接使用它

```typescript
@Get()
async findAll(@User() user: string) {
  console.log(user)
  return this.catsService.findAll()
}
```

详细内容可以参考官方文档：[自定义装饰器](https://docs.nestjs.com/custom-decorators)
## TypeOrm

Nest 与数据库无关，允许您轻松地与任何 SQL 或 NoSQL 数据库集成。根据您的喜好，您有许多选项可供您使用。在最一般的层面上，将 Nest 连接到数据库只是为数据库加载适当的 Node.js 驱动程序，就像使用 Express 或 Fastify 一样。在此我们使用 TypeORM。以下是关于 TypeORM 的一些基本信息：

- TypeORM 是一个 ORM 框架，它可以运行在 NodeJS、浏览器、Cordova、PhoneGap、Ionic、React Native、Expo 和 Electron 平台上，可以与 TypeScript 和 JavaScript (ES5, ES6, ES7, ES8) 一起使用。

### 安装

```npm
npm install --save @nestjs/typeorm typeorm mysql
```

安装完成后在 app.module.ts 中引入

```typescript
import { Module } from '@nestjs/common'
import { AppController } from './app.controller'
import { AppService } from './app.service'
import { TypeOrmModule } from '@nestjs/typeorm'
@Module({
  imports: [
    PersonModule,
    UserModule,
    TypeOrmModule.forRoot({
      // 配置信息
      type: 'mysql', //数据库类型
      username: 'root', //账号
      password: 'xiangjiayi53822', //密码
      host: 'localhost', //host
      port: 3306, //
      database: 'db', //库名
      // entities: [__dirname + '/**/*.entity{.ts,.js}'], //实体文件
      synchronize: true, //synchronize字段代表是否自动将实体类同步到数据库 生产环境不建议使用
      retryDelay: 500, //重试连接数据库间隔
      retryAttempts: 10, //重试连接数据库的次数
      autoLoadEntities: true, //如果为true,将自动加载实体 forFeature()方法注册的每个实体都将自动添加到配置对象的实体数组中
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

### 存储库

TypeORM 支持存储库设计模式，因此每个实体都有自己的存储库。可以从数据库数据源获取这些存储库。

```typescript
// user.entity.ts
// 创建一个实体，并且将其映射到数据库表
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm'

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number

  @Column()
  firstName: string

  @Column()
  lastName: string

  @Column({ default: true })
  isActive: boolean
}
```

**使用实体**

```typescript
import { Module } from '@nestjs/common'
import { TestService } from './test.service'
import { TestController } from './test.controller'
import { TypeOrmModule } from '@nestjs/typeorm'
import { Test } from './entities/test.entity'
@Module({
  imports: [TypeOrmModule.forFeature([Test])],
  controllers: [TestController],
  providers: [TestService],
})
export class TestModule {}
```

此模块使用 forFeature() 该方法定义在当前范围内注册的存储库。有了这个，我们可以将它 TestRepository 注入到使用装饰器中 TestService @InjectRepository()

```typescript
import { Injectable } from '@nestjs/common'
import { CreateTestDto } from './dto/create-test.dto'
import { UpdateTestDto } from './dto/update-test.dto'
import { Test } from './entities/test.entity'
import { InjectRepository } from '@nestjs/typeorm'
import { Repository } from 'typeorm'
@Injectable()
export class TestService {
  @InjectRepository(Test) private usersRepository: Repository<Test>
  create(createTestDto: CreateTestDto) {
    const test = new Test()
    test.firstName = createTestDto.firstName
    test.lastName = createTestDto.lastName
    test.isActive = createTestDto.isActive
    return this.usersRepository.save(test) // 添加数据
  }

  findAll(): Promise<Test[]> {
    return this.usersRepository.find()
  }

  findOne(firstName: string): Promise<Test | null> {
    return this.usersRepository.findOneBy({ firstName })
  }
  update(id: number, updateTestDto: UpdateTestDto) {
    return `This action updates a #${id} test`
  }

  async remove(id: number): Promise<void> {
    // await this.usersRepository.delete(id);
    await this.usersRepository
      .createQueryBuilder('Test')
      .delete()
      .where('id = :id', { id })
      .execute()
  }
}
```

### Repository

使用 EntityManager 您可以管理（插入、更新、删除、加载等）任何实体。实体管理器就像一个地方所有实体存储库的集合,但是我们大部分都是在使用存储库 Repository 去操作对应的实体，

在 typeorm 中存在很多操作实体来操作数据库的方法，这些方法都是通过 Repository 来实现的，而内部很多查找方法都依据 findOptions 来实现的，所以我们可以通过 findOptions 来实现更多的操作。

所有存储库和管理器 .find\* 方法都接受特殊选项，可用于查询所需的数据

```typescript
userRepository.find(findoptions: FindManyOptions<Test>): Promise<Test[]>
// 在此列出常用findoptions的查找选项
{
  select:['id','firstName'],
   // SELECT "firstName", "lastName" FROM "user
  relations:['test'],
   // LEFT JOIN "test" "test" ON "test"."id"="user"."testId"
  where:{firstName:'xiang',}, // {expression} | [{expression}]
  // WHERE "user"."firstName" = 'xiang'
 /*
 where: [
        { firstName: "Timber", lastName: "Saw" },
        { firstName: "Stan", lastName: "Lee" },
    ],
  SELECT * FROM "user" WHERE ("firstName" = 'Timber' AND "lastName" = 'Saw') OR ("firstName" = 'Stan' AND "lastName" = 'Lee')
  */
  order:{firstName:'ASC',lastName:'DESC'},
  // SELECT * FROM "user"ORDER BY "name" ASC, "id" DESC

  // 类似于limit
  skip: 5, // 跳过前5条数据
  take: 10, // 取10条数据
  //  skip 并 take 应一起使用
  cache :true  //  启用或禁用查询结果缓存。

  // 运算符
  title: Not("About #1"),
  // SELECT * FROM "post" WHERE "title" != 'About #1'
  likes: LessThan(10)
  // SELECT * FROM "post" WHERE "likes" < 10

}
```

更多请访问https://typeorm.io/find-options

## Repository API

使用 Repository 可以实现对数据库的增删改查，里面有很多方法
我观察发现带有 by 的方法是没有 by 的简写，不带 by 的需要加入 where 条件，而带 by 的不需要加入 where 条件

具体访问 https://typeorm.io/repository-api
## Nestjs 技巧

**虽然 nestjs 的框架已经很完善了，但是我们还是可以通过一些技巧来提高我们的开发效率，以下是我收集以及个人的总结**


### 1.使用 applyDecorators 减少对装饰器的依赖

对于一个业务模块，写接口以及接口文档如果不做封装，那么代码中会有大量的装饰器出现，导致代码难以阅读，所以我们可以对装饰器进行封装，减少对装饰器的依赖。

```typescript
import { applyDecorators, SetMetadata, Type } from '@nestjs/common'
import {
  ApiOperation,
  ApiOkResponse,
  ApiBadRequestResponse,2
} from '@nestjs/swagger'

export function SwaggerDocumentation(
  summary: string,
  okDescription: string,
  badRequestDescription: string,
  type: Type
) {
  return applyDecorators(
    ApiOperation({ summary }),
    ApiOkResponse({ description: okDescription, type }),
    ApiBadRequestResponse({ description: badRequestDescription })
  )
}
```

当然，这只是一个简单的例子，我们可以根据自己的业务需求来封装，这样我们就可以在接口中直接使用这个装饰器，而不用再引入大量的装饰器了。
### 2.使用缓存

缓存是提高应用程序性能的有效技术。NestJS 使用流行的缓存库（如 Redis 和 Memcached）为缓存提供内置支持。下面是如何在 NestJS 中使用缓存的示例：

```typescript
@Injectable()
export class UserService {
  constructor(private readonly cacheManager: CacheManager) {}
  async getUser(id: number): Promise<User> {
    const user = await this.cacheManager.get(`user-${id}`)
    if (user) {
      return user
    }
    const newUser = await userRepository.findOne(id)
    await this.cacheManager.set(`user-${id}`, newUser)
    return newUser
  }
}
```

在本例中，UserService 的 getUser 方法使用 CacheManager 实例缓存用户数据。如果用户数据已经在缓存中，该方法将立即返回该数据。如果没有，该方法将从数据库中检索用户数据，使用 CacheManager 的 set 方法对其进行缓存，并将其返回。

### 3.使用压缩

压缩是提高应用程序性能的另一种技术。NestJS 使用流行的压缩库（如 zlib）为压缩提供内置支持。以下是如何在 NestJS 中使用压缩的示例：

```typescript
const app = await NestFactory.create(AppModule)
app.use(compression())
await app.listen(3000)
```

在本例中，使用应用实例的使用方法将 compression 中间件添加到 NestJS 应用程序中。该中间件在将响应数据发送到客户端之前对其进行压缩，这有助于减少数据大小并缩短响应时间。

### 4.使用速率限制

为了避免用户不要频繁请求数据，以及防止网站恶意攻击，我们可以使用@nestjs/throttler 来限制用户的请求速率。

```npm
npm install --save @nestjs/throttler
```

然后在 app.module.ts 中引入 ThrottlerModule

```typescript
 imports: [
    ThrottlerModule.forRoot([
      {
        ttl: 6000, // time-to-live 每个请求计数记录的生存时间（秒）
        limit: 3, // 给定时间段内允许的最大请求数
      },
    ]),
  ],

  providers: [
    {
      provide: APP_GUARD,
      useClass: ThrottlerGuard, //这里是全局使用，实际开发需要视情况而定
    },
  ],
```

更多信息请访问https://docs.nestjs.com/security/rate-limiting#rate-limiting

### 5.使用相对路径而非绝对路径

您可以使用绝对路径或者相对路径来导入一个 es6 模块。在开发阶段他们会工作的很顺利，但是当尝试去构建它的时候这会崩溃。当您使用 Nest.js 的时候请始终使用相对路径。你稍后会感谢我的。（npm 运行 build 时，预计会出现绝对路径错误）

```typescript
// 相对
import { SecurityService } from '../security/security.service'
import { CommentService } from '../comment/comment.service'
// 绝对
import { SecurityService } from 'src/security/security.service'
import { CommentService } from 'src/comment/comment.service'
```

### 6.使用 Exclude()来隐藏数据

当您从数据库取得数据时，通常要通过 transformers 来筛选数据。主要的目的是为了删除或格式化从数据库过来的数据。这会产生很多垃圾逻辑。

```typescript
import { Exclude } from 'class-transformer'

export class UserEntity {
  id: number
  firstName: string
  lastName: string

  @Exclude()
  password: string
  // password不会出现在返回的数据中
  constructor(partial: Partial<UserEntity>) {
    Object.assign(this, partial)
  }
}
```

### 7.使用实体 getter

有些逻辑会直接访问您的实体属性。最常见的用例与密码哈希和获取全名有关。但是要注意不要用大量的逻辑来重载实体。为此使用自定义存储库

```typescript
import { Exclude } from 'class-transformer'

export class UserEntity {
  id: number
  firstName: string
  lastName: string

  get fullName() {
    return this.firstName + ' ' + this.lastName
  }
}
```

### 8.使用集中式命名导出

假设我们一个 dto 文件内部有很多的 dto，我们可以使用集中式命名导出来导出这些 dto，这样我们就可以在其他文件中直接使用这些 dto 了。只需要在 dto 文件中添加 index.dto.ts 文件，然后在 index.dto.ts 文件中导出所有的 dto 即可。

```typescript
// index.ts内部
export * from './createPost.dto'
export * from './editPost.dto'
export * from './editPostCategory.dto'
export * from './editPostStatus.dto'

// 从一个文件夹导入
import { CreatePostDto, EditPostDto } from './dto'

// 代替多个文件夹导入
import { CreatePostDto } from './dto/createPost.dto'
import { EditPostDto } from './dto/editPost.dto'
```
## nest-cli.json 配置

### nest-cli.json 配置

```json
{
  "collection": "@nestjs/schematics",
  "generateOptions": {
    "spec": false
  }, // 生成文件时不生成spec文件
  "sourceRoot": "src",
  "compilerOptions": {
    "assets": ["**/*.graphql", "**/*.hbs"]
  }
}
```
