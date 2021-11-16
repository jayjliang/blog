## 前言
最近利用空余时间在阅读Theia源码，本篇文章希望帮助大家熟悉inversify以及在Theia中的应用，让大家对Theia不要因为inversify从入门到放弃。 

[Theia](https://theia-ide.org) 相信很多人都知道，它是一个开源的IDE，类似vscode，但是他的定制化能力更强（具体还没有深入研究），所以很多有强定制化的需求的IDE很多都是基于Theia进行二次开发的。

而在阅读Theia源码的时候，有一个绕不过去的坎就是inversify，背后其实就是我们现在经常会听到的IOC（控制反转），具体包括依赖注入（DI）和依赖查询。现在使用IOC的项目越来越多了，从一开始的Angular到现在很多服务端项目（[nest.js](https://nestjs.com/)、[midwayjs](https://midwayjs.org/)），其实vscode本身也是深入依赖IOC的，只不过它是自己实现注入和解析的，而Theia的IOC则是基于inversify。

## inversify的用法

inversify的用法其实看官网介绍就能入门，这里我简化一下官网的demo让大家体会一下：

```typescript
// 先声明几个可注入的类
@injectable()
class Katana {
  public hit() {
    return "cut!";
  }
}

@injectable()
class Ninja {
  @inject(Katana)
  public katana: Katana;
  
  public fight() { return this.katana.hit(); }
}

// 初始化一个容器来承载这些可注入的类
const container = new Container();
// 向容器中注入类
container.bind<Katana>(Katana).to(Katana).inSingletonScope();
container.bind<Ninja>(Ninja).to(Ninja).inSingletonScope();
// 获取类
const ninja = container.get(Ninja);
// 输入结果是cut!
console.log(ninja.fight());
```

其实简单理解就是有一个容器来管理所有的可注入的内容，然后使用者就直接使用装饰器语法就能获取到可注入的内容，比如上面的例子就是可注入的是类，获取到的是类实例。这样使用者就不用关心类如何实例化以及如何保证单一实例等。

接下来我们学习一些常见的用法（例子来源于Theia）。

### scope
何为scope，其实就是决定你每次拿到的实例是单例还是怎样的，具体信息可以参考[官方文档](https://github.com/inversify/InversifyJS/blob/master/wiki/scope.md)，总结一下如下：

scope支持以下三种模式：
- Transient：每次从容器中获取的时候(也就是每次请求)都是一个新的实例
- Singleton：每次从容器中获取的时候（也就是每次请求）都是同一个实例
- Request：社区里也称为Scoped模式，每次请求的时候都会获取新的实例，如果在这次请求中该类被require多次，那么依然还是用同一个实例返回

```typescript
bind(MonacoCommandService).toSelf().inTransientScope(); // Default
bind(VSXEnvironment).toSelf().inRequestScope();
bind(VSXRegistryAPI).toSelf().inSingletonScope();
```

### bind
正常情况下我们使用bind的方法如下：

```js
container.bind<Katana>(Katana).to(Katana).inSingletonScope();
```
但是像Theia这种大型项目，里面会有几百个bind，我们每次都按照上面的方法来写虽然也可以，但是官方提供了[container modules](https://github.com/inversify/InversifyJS/blob/master/wiki/container_modules.md)的用法来支持管理复杂的bind(虽然我暂时也没get到太大的区别)

```typescript
const frontendApplicationModule = new ContainerModule((bind, unbind, isBound, rebind) => {
    bind(NoneIconTheme).toSelf().inSingletonScope();
    bind(LabelProviderContribution).toService(NoneIconTheme);
    bind(IconThemeService).toSelf().inSingletonScope();
    // ...
}
```

### Symbols
实际上bind语法的参数既支持字符串，也支持我们传入类

```typescript
// 类
container.bind<Katana>(Katana).to(Katana).inSingletonScope();
// 字符串
container.bind('Katana').to(Katana).inSingletonScope();
```
如果我们用字符串，那就会遇到命名空间的问题，所以这里可以使用Symbols来解决冲突问题

```typescript
const TYPES = {
    Katana: Symbol.for("Katana")
};
container.bind(TYPES.Katana).to(Katana).inSingletonScope();
```

### toSelf

这个就是一句语法糖，看下面就知道了
```typescript
bind(WidgetManager).toSelf().inSingletonScope();
// 等于
bind(WidgetManager).to(WidgetManager).inSingletonScope();
```

### toDynamicValue
绑定为动态值，在获取的时候会去执行对应的函数

```js
bind(WidgetFactory).toDynamicValue(context => ({
    id: CALLHIERARCHY_ID,
    createWidget: () => createHierarchyTreeWidget(context.container)
}));
```

### toFactory
绑定为工厂函数，与刚才的动态值不一样，动态值会执行完动态函数返回值，而工厂函数则会返回一个高阶函数，允许你进一步定制值

```typescript
bind(LoggerFactory).toFactory(ctx =>
    (name: string) => {
        const child = new Container({ defaultScope: 'Singleton' });
        child.parent = ctx.container;
        child.bind(ILogger).to(Logger).inTransientScope();
        child.bind(LoggerName).toConstantValue(name);
        return child.get(ILogger);
    }
);
```

### toService
绑定为一个服务，让其解析为以前声明过的别的类型绑定，这个绑定很特殊，没有别的任何后续操作，因为它没有返回值，看代码就知道：

```typescript
bind(FrontendApplicationContribution).toService(IconThemeApplicationContribution);

public toService(service: string | symbol | interfaces.Newable<T> | interfaces.Abstract<T>): void {
    this.toDynamicValue(
        (context) => context.container.get<T>(service)
    );
}
```

### Tagged bindings & Named bindings

这两个的作用就是你可能会将很多类绑定到一个id下面，但是这样就会出现重复的问题，也就是获取的时候不知道取哪一个，那就可以通过@tagged和@named来解决，形象理解就是打tag或者取别名。但是二者区别没有深入研究，Theia里面只使用了@named。

```typescript
// packages/task/src/browser/task-service.ts
@injectable()
export class TaskService implements TaskConfigurationClient {
    @inject(ILogger) @named('task')
    protected readonly logger: ILogger;
}

// packages/task/src/common/task-common-module.ts
function createCommonBindings(bind: interfaces.Bind): void {
    bind(ILogger).toDynamicValue(ctx => {
        const logger = ctx.container.get<ILogger>(ILogger);
        return logger.child('task');
    }).inSingletonScope().whenTargetNamed('task');
}
```



## inversify在Theia中的应用
看完上面的inversify的常见用法后，阅读Theia代码中相关的内容基本上没有问题了，但是在实际过程中还是会遇到一些相对高级的用法，接下来就从一个实际的例子来展开。

```typescript
// packages/core/src/browser/widget-manager.ts
export const WidgetFactory = Symbol('WidgetFactory');
export interface WidgetFactory {
    readonly id: string;
    createWidget(options?: any): MaybePromise<Widget>;
}

// packages/plugin-ext/src/main/browser/plugin-ext-frontend-module.ts
bind(WidgetFactory).toDynamicValue(({ container }) => ({
    id: PLUGIN_VIEW_DATA_FACTORY_ID,
    createWidget: (identifier: TreeViewWidgetIdentifier) => {
       // ...
    }
})).inSingletonScope();
bind(WidgetFactory).toDynamicValue(({ container }) => ({
    id: PLUGIN_VIEW_FACTORY_ID,
    createWidget: (identifier: PluginViewWidgetIdentifier) => {
        // ...
    }
})).inSingletonScope();

bind(WidgetFactory).toDynamicValue(({ container }) => ({
    id: PLUGIN_VIEW_CONTAINER_FACTORY_ID,
    createWidget: (identifier: ViewContainerIdentifier) =>
        container.get<ViewContainer.Factory>(ViewContainer.Factory)(identifier)
})).inSingletonScope();
```

这里可以看到，我们对WidgetFactory这个id进行了多次bind，同时我们并没有使用named bindings，那这种写法在用的时候岂不是就会报错？带着这个疑问我们看看Theia是怎么解决的。
首先我们看用的地方是怎么用的，这样才能找到突破口，代码如下：

```typescript
// packages/core/src/browser/widget-manager.ts

/**
 * The {@link WidgetManager} is the common component responsible for creating and managing widgets. Additional widget factories
 * can be registered by using the {@link WidgetFactory} contribution point. To identify a widget, created by a factory, the factory id and
 * the creation options are used. This key is commonly referred to as `description` of the widget.
 */
@injectable()
export class WidgetManager {
    @inject(ContributionProvider) @named(WidgetFactory)
    protected readonly factoryProvider: ContributionProvider<WidgetFactory>;
    
    protected get factories(): Map<string, WidgetFactory> {
        if (!this._cachedFactories) {
            this._cachedFactories = new Map();
            for (const factory of this.factoryProvider.getContributions()) {
                if (factory.id) {
                    this._cachedFactories.set(factory.id, factory);
                } else {
                    this.logger.error('Invalid ID for factory: ' + factory + ". ID was: '" + factory.id + "'.");
                }
            }
        }
        return this._cachedFactories;
    }

}

```
可以看到，和WidgetFactory相关的代码，除去bind的代码，类似inject的方法竟然只有这里，使用了factoryProvider的也只有factories方法，那我们猜测核心逻辑就是在`getContributions`方法里面，接着去找这个方法实现：

```typescript
// packages/core/src/common/contribution-provider.ts

// 只是定义
export const ContributionProvider = Symbol('ContributionProvider');

export interface ContributionProvider<T extends object> {
    getContributions(recursive?: boolean): T[]
}

// 一个实际的实现
class ContainerBasedContributionProvider<T extends object> implements ContributionProvider<T> {
    constructor(
        protected readonly serviceIdentifier: interfaces.ServiceIdentifier<T>,
        protected readonly container: interfaces.Container
    ) { }
}

// 这里很关键，使用了named bindings，所以factoryProvider最终拿到的是ContainerBasedContributionProvider的实例
export function bindContributionProvider(bindable: Bindable, id: symbol): void {
    const bindingToSyntax = (Bindable.isContainer(bindable) ? bindable.bind(ContributionProvider) : bindable(ContributionProvider));
    bindingToSyntax
        .toDynamicValue(ctx => new ContainerBasedContributionProvider(id, ctx.container))
        .inSingletonScope().whenTargetNamed(id);
}

```

可以看到`ContributionProvider`其实是一个id，里面使用了named bindings，那`WidgetManager.factoryProvider`其实最终拿到的就是`ContainerBasedContributionProvider`的实例，那接下来看看`getContributions`的实现

```typescript
class ContainerBasedContributionProvider<T extends object> implements ContributionProvider<T> {
    protected services: T[] | undefined;
    getContributions(recursive?: boolean): T[] {
        if (this.services === undefined) {
            const currentServices: T[] = [];
            let currentContainer: interfaces.Container | null = this.container;
            // eslint-disable-next-line no-null/no-null
            while (currentContainer !== null) {
                if (currentContainer.isBound(this.serviceIdentifier)) {
                    try {
                        currentServices.push(...currentContainer.getAll(this.serviceIdentifier));
                    } catch (error) {
                        console.error(error);
                    }
                }
                // eslint-disable-next-line no-null/no-null
                currentContainer = recursive === true ? currentContainer.parent : null;
            }
            this.services = currentServices;
        }
        return this.services;
    }
}
```

看到这里大家应该就明白了，我们对WidgetFactory进行了多次bind，虽然没办法直接使用inject来注入使用，但是我们可以使用`Container.getAll`来获取所有注入的类，并且把所有的类保存下来，然后根据id存储到Map里面，最后使用的时候直接根据id取出对应的类来用就好啦，具体可以看看`WidgetManager.getOrCreateWidget`方法。

ContributionProvider在很多地方都有使用，底层的思路都是类似的。
## 总结
Theia对inversify的使用是非常深度且频繁的，希望读完这篇文章能帮大家阅读Theia源码的时候有一些帮助。

这里也想表达一下自己的一些看法。使用IOC本质上是好的，但是个人感觉Theia是不是有一些过度使用了，基本上所有的模块都是基于IOC来实现的，这样的好处是保持一致性以及拓展性，例如插件可以向主模块bind一些能力或者进行一些拓展，但是感觉缺点也很明显，就是代码比较晦涩，流程不够清晰。

一些真实需要单例的类比较适合使用IOC，但是一些功能模块或者功能函数也用IOC感觉就有点大材小用了，同时增加了理解成本。欢迎大家也对这一点进行讨论，看看是我们的代码能力以及架构能力不够，还是他们真的有点过度使用～

## 参考
[带你学习inversify.js系列 - inversify基础知识学习](https://zhuanlan.zhihu.com/p/137542149)
