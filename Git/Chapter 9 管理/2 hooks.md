# hooks

## Git Hooks是什么

Git Hooks可以在运行各个动作的前后自动去触发某个脚本来做前置处理或是后置处理，这个功能很常被用来在commit前自动检查代码的排版或是写法上的问题，确保commit过的代码排版和风格是一致的。

Git Hooks又分为在客户端(client-side)触发的Hook，以及在服务器端(server-side)触发的Hook。

## Hooks目录结构

当我们运行`git init`指令初始化一个新版本库时，项目目录底下会多出一个`.git`目录，这个目录用来存放Git设置与版本控制所需的文件，它并不能被add进Git仓库中。Git 默认会在`.git/hooks`目录中放置一些示例脚本，hooks目录钟存放了很多sample文件，这里面的每一个sample，其实就是每一个git钩子：

```
.GIT\HOOKS
    applypatch-msg.sample
    commit-msg.sample
    fsmonitor-watchman.sample
    post-update.sample
    pre-applypatch.sample
    pre-commit.sample
    pre-merge-commit.sample
    pre-push.sample
    pre-rebase.sample
    pre-receive.sample
    prepare-commit-msg.sample
    push-to-checkout.sample
    update.sample
```

所有的示例都是 shell 脚本，其中一些还混杂了 Perl 代码，不过，任何正确命名的可执行脚本都可以正常使用 —— 你可以用 Ruby 或 Python，或任何你熟悉的语言编写它们。这些示例的名字 都是以 .sample 结尾，如果你想启用它们，得先移除这个后缀。把一个正确命名(不带扩展名)且可执行的文件放入`.git/hooks`目录中，即可激活该钩子脚本。这样一来，它就能被 Git 调用。

每个脚本在被触发运行时，各自会有不同的参数传入。在`.git/hooks`目录中，会有以`.sample`结尾的Shell脚本文件，会说明该Git Hooks脚本应会被传入什么样的参数，以及它该根据什么样的状况回传什么exit status，您也可以查阅[官方文档](https://git-scm.com/book/zh-tw/v2/Customizing-Git-Git-Hooks)获得更详细的说明。

> 要被用作Git Hooks脚本的文件必须具有可运行权限才可以被触发运行。

## 常用hooks钩子

| Git Hook           | 调用时机                             | 说明                                                  |
| ------------------ | ------------------------------------ | ----------------------------------------------------- |
| pre-applypath      | git am执行前                         |                                                       |
| applypath-msg      | git am执行前                         |                                                       |
| post-applypath     | git am执行后                         | 不影响git am结果                                      |
| pre-commit         | git commit执行前                     | 可以用git commit --no-verify绕过                      |
| cmmit-msg          | git commit执行前                     | 可以用git commit --no-verify绕过                      |
| post-commit        | git commit执行后                     | 不影响git commit的结果                                |
| pre-merge-commit   | git merge执行前                      | 可以用git merge --no-verify绕过                       |
| prepare-cmmit-msg  | git commit执行后，编辑器打开前       |                                                       |
| pre-rebase         | git rebase执行前                     |                                                       |
| post-checkout      | git checkout或git switch执行后       | 如果不使用–no-checout参数，则在git clone 之后也会执行 |
| post-merge         | git commit执行后                     | 在执行git pull时也会被调用                            |
| pre-push           | git push执行前                       |                                                       |
| pre-receive        | git-receive-pack 执行前              |                                                       |
| post-rewrite       | git commit --amend或git rebase执行前 |                                                       |
| sendemail-validate | git send-email执行前                 |                                                       |
| update             | git receive-pack执行后               |                                                       |
| post-receive       | git receive-pack执行后               | 不影响git-receive-pack的结果                          |

## 常见钩子

### pre-commit

pre-commit 钩子在键入提交信息前运行。它用于检查即将提交的快照，例如，检查是否有所遗漏，确保测试运行，以及核查代码。如果该钩子以非零值退出，Git 将放弃此次提交。你可以利用该钩子，来检查代码风格是否一致(运行类似 lint 的程序)、尾随空白字符是否存在或新方法的文档是否适当。

### pre-merge-commit

pre-merge-commit钩子由git-merge调用，在合并操作执行成功，获得提交消息之前执行该钩子。该钩子不接收任何参数，如果脚本以非0值退出，将中止合并操作。如果在合并过程中出现了冲突，且处理冲突时并不是一次处理完成，而是分为了多个commit，这种情况下该钩子不会执行。

### prepare-commit-msg

prepare-commit-msg 钩子在启动提交信息编辑器之前，默认信息被创建之后运行。它允许你编辑提交者所看到的默认信息。该钩子接收一些选项:存有当前提交信息的文件的路径、提交类型和修补提交的提交的 SHA-1 校验。它对一般的提交来说并没有什么用;然而对那些会自动产生默认信息的提交，如提交信息模板、合并提交、压缩提交和修订提交等非常实用。你可以结合提交模板来使用它，动态地插入信息。

### commit-msg

commit-msg 钩子接收一个参数，此参数即上文提到的，存有当前提交信息的临时文件的路径。如果该钩子脚本以非零值退出，Git 将放弃提交，因此，可以用来在提交通过前验证项目状态或提交信息。

### post-commit

post-commit 钩子在整个提交过程完成后运行。它不接收任何参数，但你可以很容易地通过运行 git log -1 HEAD 来获得最后一次的提交信息。该钩子一般用于通知之类的事情。

## 保存与共享Git Hooks

由于Git Hooks脚本缺省存放在无法被`git add`的`.git`目录下，因此如果要能使它被push到远程的Git仓库上，就需要换个位置放，而不是放在`.git/hooks`。

笔者自己是习惯在项目目录底下再创建一个`.githooks`目录，并把Git Hooks脚本都放在这个目录下。

当然，我们还是要让Git知道Git Hooks脚本放在哪才行，可以运行如下的指令：

```shell
git config core.hooksPath .githooks
```

用如上的指令将Git的`core.hooksPath`字段设置为Git Hooks脚本所在的目录即可。注意这个设置也只会在目前的这个本地端项目有效，并不会被push到远程的Git仓库上。

我们可以在`CONTRIBUTING.md`或`README.md`中撰写说明，让刚clone这个Git项目的开发者主动去运行这个指令。也可以把这个指令写在程序项目自动化脚本的某个位置。

不过有的环境里可能会没有`git`指令工具，此时`git config`指令会运行失败，而导致后续原本要跑的流程中止了。将指令改成以下这样，再加进程式项目里应该会比较好一点：

```shell
git config core.hooksPath .githooks || exit 0
```

## 如何跳过钩子？

有些钩子可以根据返回值决定是否继续执行后续动作，这些钩子是可以跳过的。只需要在执行命令时添加 --no-verify 可以跳过钩子的执行。
