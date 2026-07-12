# Horizon 视觉组 Github 组织代码管理规范 V1.0

##### by ZYL

## 一、成员本地仓库设置和操作流程

### 1. 成员仓库初始化流程
```bash
# 克隆仓库
git clone https://github.com/组织名/仓库名.git
cd 仓库名

# 查看可用分支
git branch -r

# 切换到自己的分支（以成员sentry为例）
#这一步很重要，不要随意在main分支提交内容
git checkout sentry

#查看当前所属的分支（带*号的为当前所在分支）
git branch

# 推送到远程并建立关联
git push -u origin sentry
```

### 2. 成员日常开发流程
```bash
# 1. 开始新功能前，同步主分支最新代码（先查看一下main是否有更新）
git checkout main
git pull origin main

# 2. 切换到自己的开发分支
git checkout sentry

#3.并合并最新main（可根据实际需要决定）
git merge main
# 或使用rebase（保持线性历史）
git rebase main

# 3. 进行开发工作...
# 编辑代码和日志

# 4. 提交更改（先查看当前自己处于哪个分支）
git status
git add .
git commit -m "COMMIT名称"

# 5. 推送到自己的远程分支
git push origin sentry
```

### 3. 成员申请合并到main分支

1. 在GitHub上创建Pull Request：
   - Base: `main`
   - Compare: `sentry`
2. 填写PR描述，请求审查
3. 等待作为管理员审查代码，可以：
   - 直接合并
   - 请求修改
   - 添加评论

### 4.成员的同步操作：
```bash
# 1. 当PR被合并后，同步最新main到自己的分支
git checkout main
git pull origin main

git checkout sentry
git rebase main
# 或
git merge main

# 2. 如果有冲突，解决冲突
git add .
git rebase --continue
# 或
git commit -m "Merge main"

# 3. 推送更新后的分支
git push origin sentry
# 如果使用了rebase可能需要强制推送（确保只有你使用该分支）
git push -f origin sentry
```

## 二、命名格式规范

### 1. 主线分支开发日志文件:

位置：在项目根目录下的DEV_LOG.MD

### 2.分支日志编写规范：

位置：在项目根目录下创建BR_DEV_LOG.MD

**示例模板：**

```
# sentry分支开发日志
### C1：
- 根版本: main-M1（基于主线哪个版本修改）
- 功能更新: 添加xxxx（feature和test）
- 质量更新：修复xxxx（fix和docs）
- 已知问题：xxxx
```

### 4. 分支Commit名称规范

**临时commit：**单一功能或较小规模的修改，或对上一个版本临时的修复，用于临时测试或含风险修改前用于保存状态，可以暂时不更新日志，但在下次正式commit时必须补充所有内容修改的日志

**命名规则：**类型-自定义内容（日期/时间/其他描述）

```
例如：feature-260109C1
```

**正式commit：**指该commit下的功能已经经过测试，并已经编写好相对上一个正式commit的差异的文档，可以用于将内容合并到主线分支，正式commit之前必须完成日志内容更新

**命名：**车组名-C+数字

```
例如：sentry-C1
```

**类型说明**:

- `feature`: 新功能
- `fix`: bug修复
- `docs`: 文档
- `test`: 测试



## 三、常见问题解决

### 1. 成员push被拒绝
```bash
# 检查权限
git push origin 分支名
# 如果失败，确认：
# 1. 是否有该分支的push权限
# 2. 分支是否受保护
# 3. 是否在正确分支
```

### 2. 合并冲突解决
```bash
# 查看冲突文件
git status

# 手动解决冲突后
git add .
git rebase --continue  # 如果使用rebase
# 或
git commit -m "解决冲突"
```

### 3. 误操作恢复
```bash
# 撤销最近一次提交
git reset --soft HEAD~1

# 丢弃工作区修改
git checkout -- 文件名
# 或全部丢弃
git checkout -- .

# 从远程恢复分支
git fetch origin
git checkout -b 分支名 origin/分支名 --force
```

**注意：**定期从main分支合并/rebase到自己的分支，避免分支差异过大导致合并困难。
