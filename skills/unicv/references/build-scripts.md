# 打包脚本

## build.mjs 核心逻辑

交互式三步构建流程：

```
Step 1/3：写入对应环境的 appid 到 src/manifest.json
Step 2/3：执行 uni build -p mp-weixin --mode {env}（pnpm 优先，失败回退 npm）
Step 3/3：调用微信开发者工具 CLI 打开产物目录
```

失败时自动回滚 manifest.json 的 appid。

## build.lib.mjs 工具函数说明

```javascript
// ENV_CONFIG 配置结构
const ENV_CONFIG = {
  test: {
    appid: 'wx你的测试appid',
    label: '测试环境',
    mode: 'test'
  },
  prod: {
    appid: 'wx你的生产appid',
    label: '生产环境',
    mode: 'prod'
  }
};

// writeAppid(manifestPath, newAppid) → 返回旧 appid（用于回滚）
// runBuildCommand({ environment, projectDir, packageManagers }) → 执行构建
// openWechatIDE({ projectDir, buildOutputPath }) → 打开微信开发者工具
// validateNodeVersion({ expectedVersion, currentVersion }) → 检查 Node 版本
// probePackageManagers() → { primary: 'pnpm', fallback: 'npm' }
// readExpectedNodeVersion(projectDir) → 从 .nvmrc 读取期望版本
// resolveEnvironment({ isTTY, requestedEnvironment, envValue, promptForEnvironment })
// resolveOpenIDE({ isTTY, requestedOpen, envValue })
```

## build.mjs CLI 参数

```bash
# 指定环境（跳过交互提示）
node build.mjs --env=test
node build.mjs --env=prod

# 控制是否打开微信开发者工具
node build.mjs --open
node build.mjs --no-open

# 环境变量方式（CI/CD 场景）
BUILD_ENV=prod OPEN_WECHAT_IDE=false node build.mjs
```

## build.mjs 完整实现参考

```javascript
// build.mjs（ES Module 格式）
import { fileURLToPath } from 'url';
import { dirname, join } from 'path';
import symbols from 'log-symbols';
import spin from 'io-spin';
import chalk from 'chalk';
import inquirer from 'inquirer';
import {
  ENV_CONFIG,
  openWechatIDE,
  probePackageManagers,
  readExpectedNodeVersion,
  resolveEnvironment,
  resolveOpenIDE,
  runBuildCommand,
  validateNodeVersion,
  writeAppid
} from './build.lib.mjs';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
const MANIFEST_PATH = join(__dirname, 'src/manifest.json');
const isTTY = process.stdout.isTTY;

// 日志工具
const log = {
  success: msg => console.log(symbols.success, chalk.green(msg)),
  error:   msg => console.log(symbols.error, chalk.red(msg)),
  info:    msg => console.log(symbols.info, chalk.cyan(msg)),
  warn:    msg => console.log(symbols.warning, chalk.yellow(msg))
};

async function main() {
  console.log('');
  console.log(chalk.bold.cyan('═══════════════════════════════════════'));
  console.log(chalk.bold.cyan('         你的项目名称 - 构建打包工具'));
  console.log(chalk.bold.cyan('═══════════════════════════════════════'));
  console.log('');

  // 检查 Node 版本和包管理器
  const expectedVersion = readExpectedNodeVersion(__dirname);
  validateNodeVersion({ expectedVersion, currentVersion: process.version });
  const packageManagers = probePackageManagers();
  log.info(`构建包管理器: ${packageManagers.primary}`);

  // 解析 CLI 参数
  const cliArgs = parseCliArgs(process.argv.slice(2));
  const environment = await resolveEnvironment({
    isTTY,
    requestedEnvironment: cliArgs.environment,
    envValue: process.env.BUILD_ENV,
    promptForEnvironment: async () => {
      const { env } = await inquirer.prompt([{
        name: 'env',
        type: 'list',
        message: '请选择需要打包的环境:',
        choices: [
          { name: `${symbols.success}  测试环境 (${ENV_CONFIG.test.appid})`, value: 'test' },
          { name: `${symbols.success}  生产环境 (${ENV_CONFIG.prod.appid})`, value: 'prod' }
        ],
        default: 'test'
      }]);
      return env;
    }
  });

  const { label, appid } = ENV_CONFIG[environment];
  let oldAppid = null;

  try {
    console.log(chalk.dim(`\n── Step 1/3: 写入 ${label} appid ──`));
    oldAppid = writeAppid(MANIFEST_PATH, appid);
    log.success(`appid 已更新为: ${appid}`);

    console.log(chalk.dim(`\n── Step 2/3: 编译小程序 ──`));
    log.info(`${label}正在打包，请稍后...`);
    runBuildCommand({ environment, projectDir: __dirname, packageManagers });
    log.success('微信小程序编译完成');

    console.log(chalk.dim(`\n── Step 3/3: 启动微信开发者工具 ──`));
    const shouldOpen = resolveOpenIDE({
      isTTY,
      requestedOpen: cliArgs.openIDE,
      envValue: process.env.OPEN_WECHAT_IDE
    });
    if (shouldOpen) {
      openWechatIDE({ projectDir: __dirname, buildOutputPath: 'dist/build/mp-weixin' });
      log.success('微信开发者工具已打开');
    } else {
      log.info('已跳过打开微信开发者工具');
    }

    console.log('');
    console.log(chalk.bold.green(`  ${label}打包流程完成！`));
    console.log('');
  } catch (error) {
    if (oldAppid) {
      writeAppid(MANIFEST_PATH, oldAppid);
      log.warn('已自动回滚 manifest.json 的 appid');
    }
    console.error(symbols.error, chalk.red(error instanceof Error ? error.message : String(error)));
    process.exit(1);
  }
}

function parseCliArgs(argv) {
  const args = { environment: undefined, openIDE: undefined };
  for (let i = 0; i < argv.length; i++) {
    if (argv[i] === '--open') { args.openIDE = true; continue; }
    if (argv[i] === '--no-open') { args.openIDE = false; continue; }
    if (argv[i].startsWith('--env=')) { args.environment = argv[i].split('=')[1]; continue; }
    if (argv[i] === '--env') { args.environment = argv[++i]; }
  }
  return args;
}

main();
```

## 微信开发者工具 CLI 前提条件

1. 安装微信开发者工具
2. 开启服务端口：微信开发者工具 → 设置 → 安全 → 服务端口
3. 确认 `cli` 命令路径（macOS 默认 `/Applications/wechatwebdevtools.app/Contents/MacOS/cli`）
