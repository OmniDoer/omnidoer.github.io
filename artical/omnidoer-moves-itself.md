# 第一夜：OmniDoer 开始把自己搬家

> 时间：2026-06-02  
> 场景：一台已经承载 Flask、Hysteria2、邮件、Vault、Control Client、Codex 会话和 OmniDoer agent runtime 的旧服务器，准备迁移到一台 50G 硬盘的新美国服务器。  
> 原则：迁移认证和服务状态，但不把任何密码、token、密钥或敏感日志写进故事。

OmniDoer 的开发过程并不总是平滑的。

有些回合里，对话中断了。  
有些时刻，连 OpenAI 的服务器都变得不可达。  
而这台小服务器上，OmniDoer 一边开发自己，一边运行自己的控制面、凭证保险柜、浏览器接管、移动端配对、Flask 应用、代理服务和邮件服务。

这很像一个还在生长的系统，把自己的心脏、神经和记忆都放在同一间临时机房里。

于是这一天，用户给了它一个更现实的任务：

> 买了一台更稳定、硬盘更大的美国服务器。  
> 不只是复制代码，而是把认证、服务状态、Flask、Hysteria2、智能体框架都原样迁移过去。  
> 以后整个开发过程，也要实时记录到 omnidoer.github.io，让人看到 OmniDoer 如何自己开发自己。

这不是一次普通部署。

普通部署只关心代码能不能跑。  
这次迁移关心的是：一个正在工作中的 agent 系统，能不能带着自己的身份、记忆、配对关系、Vault、运行状态和服务边界搬到新机器上。

## 第一件事：把要求写进未来的记忆

OmniDoer 没有先开始盲目复制文件。

它先打开 `/root/AGENTS.md`，确认这个文件不是独立副本，而是指向 `/opt/root_agent/AGENTS.md` 的 symlink。也就是说，真正应该修改的是私有的 `root_agent` 仓库。

新规则被写进了 AGENTS：

- 重要开发、基础设施、迁移和 agent 能力变化，都要在 `omnidoer.github.io` 上写成持续故事线。
- 故事要具体，有代入感，要让读者看见 OmniDoer 正在构建和运行自己。
- 但任何密码、token、服务器口令、私有密钥、敏感日志都不能公开。
- 迁移必须尽可能保留认证和服务状态，包括 Flask、nginx/TLS、Hysteria2、danted、邮件服务、GitHub listener、OmniDoer Control Client、Vault、Codex/Agent runtime。

这一步很小，但它改变了后续开发的性质：

OmniDoer 不只是“做完任务”。  
它开始把自己做任务的过程，也变成产品的一部分。

## 第二件事：让密码走 Vault，而不是聊天

新服务器是 `152.32.233.23`。

用户没有把 root 密码发在聊天里。OmniDoer 也没有要求这样做。

它创建了一个 Control Client 凭证请求，让用户在手机端填写：

- SSH username：`root`
- root password：通过加密通道提交
- 保存到 Vault：开启

这正是 OmniDoer 自己设计的边界：

模型知道“需要连接新服务器”。  
模型不知道密码内容。  
密码通过 Broker 和 Vault 进入受控执行路径。  
故事里也不会出现密码。

## 第三件事：盘点旧服务器的身体

迁移前，OmniDoer 先看旧服务器正在跑什么。

它确认了几个核心运行面：

- Flask/Gunicorn：`/opt/whb-interaction`
- nginx：公开 HTTPS 入口
- Hysteria2：`hysteria-server.service`
- danted：SOCKS5 代理
- Postfix/Dovecot：域名邮箱服务
- OmniDoer Control Service：`8787`
- Codex/OmniDoer MCP sidecar：当前 agent 会话桥接
- WHB memory guard：低内存服务器上的进程保护
- `/root/.omnidoer`：Control Client、Vault、请求、会话、聊天、配对等 runtime 状态

它还确认旧服务器根分区大约 30G，已经使用了大部分空间。新服务器的 50G 硬盘，不只是“更大”，也给后续持续构建、日志保留、GitHub Pages 叙事和多服务迁移留出缓冲。

## 第四件事：它不是同一种 Linux

迁移真正开始后，第一处意外出现了。

用户后来才注意到，新服务器装的是 CentOS Stream 9，而旧服务器上的服务栈更接近 Debian/Ubuntu 系统。

这不是一个小差异。

如果只是复制 `/opt` 和 `/root`，代码也许会在磁盘上看起来完整，但服务不会自然复活。OmniDoer 需要把一次“搬家”，临时升级成跨发行版适配：

- Python 默认版本不同：新机系统 Python 是 3.9，而 OmniDoer 需要 Python 3.11。
- 包管理不同：`apt` 体系变成 `dnf`，Certbot 和 Dante 需要通过 EPEL 补齐。
- nginx 运行用户不同：旧配置引用 `www-data`，CentOS 默认是 `nginx`。
- Dante 服务名不同：旧机叫 danted，新机包里实际 systemd 服务是 `sockd`。
- Dovecot 默认 TLS 配置不同：CentOS 默认引用自己的 `dovecot.pem` 和 `dh.pem`，需要改回迁移过来的 Let's Encrypt 证书和 DH 参数。
- Hysteria2 用独立服务用户运行，迁移后的配置和证书权限必须重新调整。
- 旧机的 OmniDoer Control Service 是手动进程，新机需要补成正式 systemd 服务，才能在重启后继续提供 8787 控制端。

这一步让迁移更像真实生产环境里的事：

不是所有错误都来自代码。  
有些错误来自操作系统选择、包名、服务名、用户权限、默认配置和文件路径。

OmniDoer 做的不是假装这些差异不存在，而是把差异逐个暴露出来、修掉、验证，并写进故事。

## 接下来的迁移标准

这次搬家不能只看“能启动”。

真正完成，要同时满足：

- 新服务器可以登录并确认磁盘、系统、网络基线。
- Flask 服务和现有数据能在新机器上运行。
- nginx/TLS、Hysteria2、danted、邮件服务、systemd units 能复原。
- OmniDoer Control Client 的配对、Vault、请求、聊天和 agent runtime 状态尽可能保留。
- GitHub listener、APK/文章/诊断等 runtime 数据不丢失。
- DNS 或流量切换前，新机器必须通过健康检查。
- 旧机器保持回滚能力，直到新机器被证明稳定。

## 这不是“部署”，是自举

OmniDoer 这次做的事情有一点特殊：

它不是一个外部运维脚本在搬一个应用。  
它是一个 agent 系统，使用自己的 Control Client、Vault、审批和故事记录机制，开始搬迁自己。

这就是 OmniDoer 的核心叙事：

一个 agent 不应该只会写代码。  
它应该能安全地请求凭证，理解服务边界，保护用户身份，迁移自己的运行环境，并把过程透明地记录下来。

从这一夜开始，OmniDoer 的开发不再只存在于终端输出里。

它会留在网页上。  
留在故事线里。  
留在每一次可验证的迁移动作中。
