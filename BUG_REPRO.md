# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

推理运行完成后仍有隔离快照和未决质量结论时，归档操作却成功并释放了算力池。请修复归档前置条件，确保所有快照完成质量处置后才能归档运行。 请只修改必要的生产代码，不得新增、删除或修改测试文件，不得跳过测试或放宽断言。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/ai-featuremesh-task-07
- 仓库地址：https://github.com/zhanglei10281852-gif/ai-featuremesh-task-07.git
- parent SHA：d86ebf480c56bda3122271966be76f2498cbe79c

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/ai-featuremesh-task-07.git bug-repro
cd bug-repro
git checkout --detach d86ebf480c56bda3122271966be76f2498cbe79c
go test ./internal/service -run "^TestArchivingWaitsForQualityResolution$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestArchivingWaitsForQualityResolution$" -count=1
--- FAIL: TestArchivingWaitsForQualityResolution (0.50s)
    annotation_behavior_test.go:97: close with unresolved samples error = <nil>
FAIL
FAIL	github.com/zhanglei10281852-gif/ai-featuremesh-base/internal/service	0.504s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestArchivingWaitsForQualityResolution$" -count=1
--- FAIL: TestArchivingWaitsForQualityResolution (1.28s)
    annotation_behavior_test.go:97: close with unresolved samples error = <nil>
FAIL
FAIL	github.com/zhanglei10281852-gif/ai-featuremesh-base/internal/service	1.473s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

## 通过条件

定向公开行为验证通过，相关包和全量测试通过，go vet 及 linux/amd64 构建通过。 定向命令必须由修复前失败变为修复后通过；相关包、go test ./... -count=1、go vet ./... 和 linux/amd64 构建必须通过；回退 gold 关键修改后定向命令重新失败。
