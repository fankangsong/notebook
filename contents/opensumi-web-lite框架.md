# Opensumi web lite 版本框架

仓库地址: https://github.com/opensumi/ide-startup-lite

## 环境安装

1. 安装 Python3.0
2. 安装 [node-gyp](https://github.com/nodejs/node-gyp#installation)

## 相关模块

- BrowserFS 基于浏览器的文件系统，相关文档：
  - https://github.com/jvilk/BrowserFS
  - https://jvilk.com/browserfs/2.0.0-beta/index.html
- di
  - 什么是 di (depentecy injection)
  - di 是 IOC 的一种实现
  - [前端中的 IoC 理念](https://juejin.cn/post/6844903750843236366)


## FileProviderContribution

`FileProviderContribution` 的源码文件在：

```ts
// packages\file-service\src\browser\file-service-contribution.ts

// 代码细节：

export class FileServiceContribution implements ClientAppContribution {
  @Autowired(IFileServiceClient)
  protected readonly fileSystem: FileServiceClient;

  @Autowired(IDiskFileProvider)
  private diskFileServiceProvider: IDiskFileProvider;

  @Autowired(FsProviderContribution)
  contributionProvider: ContributionProvider<FsProviderContribution>;

  constructor() {
    // 初始化资源读取逻辑，需要在最早初始化时注册
    // 否则后续注册的 debug\user_stroage 等将无法正常使用
    this.fileSystem.registerProvider(Schemes.file, this.diskFileServiceProvider);
  }

  async initialize() {
    const fsProviderContributions = this.contributionProvider.getContributions();

    await Promise.all(
      fsProviderContributions.map(async (contrib) => {
        contrib.registerProvider && (await contrib.registerProvider(this.fileSystem));
      }),
    );

    await Promise.all(
      fsProviderContributions.map(async (contrib) => {
        contrib.onFileServiceReady && (await contrib.onFileServiceReady());
      }),
    );
  }
}
```