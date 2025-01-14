# Workspace

Vitest 提供了内置支持 monorepo 的工作区配置文件。你可以创建一个工作区来定义项目设置。

## 定义 Workspace

一个工作区应该在其根目录下（如果你有配置文件，则与其位于同一文件夹中）有一个名为 `vitest.workspace` 或 `vitest.projects` 的文件。Vitest 支持 `ts`/`js`/`json` 扩展名的文件。

工作区配置文件应该有一个默认导出，其中包含一个文件列表或 glob 模式，引用你的项目。例如，如果你有一个名为 `packages` 的项目文件夹，你可以使用以下配置文件定义一个工作区：

:::code-group
```ts [vitest.workspace.ts]
export default [
  'packages/*'
]
```
:::

即使某个文件夹中没有配置文件，Vitest 也会将 `packages` 文件夹中的每个文件夹视为单独的项目。

::: warning
除非在此配置文件中指定，否则 Vitest 不会将根配置文件视为工作区项目（因此它不会运行在 `include` 中指定的测试）。
:::

你还可以使用项目的配置文件引用项目：

:::code-group
```ts [vitest.workspace.ts]
export default [
  'packages/*/vitest.config.{e2e,unit}.ts'
]
```
:::

该模式仅包括具有包含 `e2e` 和 `unit` 的 `vitest.config` 文件的项目。这些关键字需要在文件扩展名之前出现。

::: warning
如果你正在使用 glob 模式引用文件名，请确保你的配置文件以 `vite.config` 或 `vitest.config` 开头。否则，Vitest将跳过它。
:::

你还可以使用内联配置定义项目。工作区文件支持同时使用这两种语法。

:::code-group
```ts [vitest.workspace.ts]
import { defineWorkspace } from 'vitest/config'

// defineWorkspace provides a nice type hinting DX
export default defineWorkspace([
  'packages/*',
  {
    // add "extends" to merge two configs together
    extends: './vite.config.js',
    test: {
      include: ['tests/**/*.{browser}.test.{ts,js}'],
      // it is recommended to define a name when using inline configs
      name: 'happy-dom',
      environment: 'happy-dom',
    }
  },
  {
    test: {
      include: ['tests/**/*.{node}.test.{ts,js}'],
      name: 'node',
      environment: 'node',
    }
  }
])
```
:::

::: warning
所有项目应该具有唯一的名称。否则，Vitest会抛出错误。如果你没有在内联配置中提供名称，Vitest将分配一个数字。如果你没有在使用 glob 语法定义的项目配置中提供名称，Vitest将默认使用目录名称。
:::

如果你不依赖内联配置，你可以在根目录中创建一个小的 JSON 文件：

:::code-group
```json [vitest.workspace.json]
[
  "packages/*"
]
```
:::

工作区项目不支持所有配置属性。为了获得更好的类型安全性，请在项目配置文件中使用 `defineProject` 方法而不是 `defineConfig` 方法：

:::code-group
```ts [packages/a/vitest.config.ts]
import { defineProject } from 'vitest/config'

export default defineProject({
  test: {
    environment: 'jsdom',
    // "reporters" is not supported in a project config,
    // so it will show an error
    reporters: ['json']
  }
})
```
:::

## Configuration

没有任何配置选项从根级别的配置文件继承。你可以创建一个共享的配置文件，并将其与项目配置文件合并：

:::code-group
```ts [packages/a/vitest.config.ts]
import { defineProject, mergeConfig } from 'vitest/config'
import configShared from '../vitest.shared.js'

export default mergeConfig(
  configShared,
  defineProject({
    test: {
      environment: 'jsdom',
    }
  })
)
```
:::

此外，某些配置选项不允许在项目配置中使用。其中最明显的是：

- `coverage`: 覆盖率是针对整个工作区进行的。
- `reporters`: 仅支持根级别的报告器。
- `resolveSnapshotPath`: 仅支持根级别的解析器。
- 所有其他不影响测试运行器的选项。

::: tip
所有不支持在项目配置中使用的配置选项，在 ["Config"](/config/) 页面上都有一个 <NonProjectOption /> 标记。
:::

## Coverage

工作区项目的覆盖率可以直接使用。但是，如果你启用了 [`all`](/config/#coverage-all) 选项并且在某些项目中使用了非常规扩展名，则需要在根配置文件中使用处理此扩展名的插件。

例如，如果你有一个使用 Vue 文件的包，并且它有自己的配置文件，但是某些文件没有在测试中导入，覆盖率分析将会失败，因为它依赖于根配置而不是项目配置，试图分析未使用文件的使用情况。
