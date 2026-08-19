# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

同一个出车请求重复提交时，第二次请求没有返回第一次创建的行程，而是再次尝试改变班次状态并报冲突。请修复幂等重放行为。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/t63-qa-23
- 仓库地址：https://github.com/zhanglei10281852-gif/t63-qa-23.git
- parent SHA：3ff5b57071f9b42f3e63cecab758489fde4f672e

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/t63-qa-23.git bug-repro
cd bug-repro
git checkout --detach 3ff5b57071f9b42f3e63cecab758489fde4f672e
go test ./internal/httpapi -run TestCompleteDispatchFlowIsTransactionalAndIdempotent -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/httpapi -run TestCompleteDispatchFlowIsTransactionalAndIdempotent -count=1
FAIL	sanitation-operations/internal/httpapi [build failed]
FAIL

```

stderr：

```text
warning: internal/httpapi/server_test.go has type 100755, expected 100644
warning: internal/httpapi/server_test.go has type 100755, expected 100644
# sanitation-operations/internal/service/dispatch
internal/service/dispatch/service.go:115:6: no new variables on left side of :=

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/httpapi -run TestCompleteDispatchFlowIsTransactionalAndIdempotent -count=1
FAIL	sanitation-operations/internal/httpapi [build failed]
FAIL

```

stderr：

```text
warning: internal/httpapi/server_test.go has type 100755, expected 100644
warning: internal/httpapi/server_test.go has type 100755, expected 100644
# sanitation-operations/internal/service/dispatch
internal/service/dispatch/service.go:115:6: no new variables on left side of :=

```

## 通过条件

在触发条件下，定向测试 TestCompleteDispatchFlowIsTransactionalAndIdempotent 应通过，相关包、全量测试、竞态测试和构建检查均通过；回退 gold 唯一修复后定向测试重新失败。
