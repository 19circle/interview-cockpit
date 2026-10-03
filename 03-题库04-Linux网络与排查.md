# 题库04 · Linux网络与排查

## 模块说明

**考频**：Linux + 网络是测试开发岗的"基本功分水岭"。深圳测开面试几乎必问：日志排查、网络协议（TCP/HTTP）、容器（Docker/k8s）。常和技术深挖结合，比如"服务挂了你怎么定位"。

**你的优势与短板**：
- 优势：银企直联 40+ 银行渠道，Linux/k8s 日志排查是日常；独立开发 FastAPI 模块自己写 Dockerfile、用 Docker 部署；环境信息模块用 JWT 鉴权，对 Cookie/Session/Token 有开发级理解；Fiddler 抓包、Postman/JMeter 都是常用工具。
- 短板：网络底层（TCP 状态机细节、OSI 七层）偏理论，需把"用过"讲成"能说清原理"；k8s 偏使用排查，集群运维深度有限。

**怎么用**：Linux 命令题动手敲一遍（别只背）；网络题理解"为什么"；Docker/k8s 题结合你部署 FastAPI 的真实经历讲。

---

## 一、高频题（30题）

### 1. 测试为什么要会 Linux？
【考点】岗位匹配。
【参考答案】测开的服务基本跑在 Linux 上（应用服务器、数据库、k8s 节点），测试环境、预发、生产都在 Linux。不会 Linux 就看不到真实日志、查不了进程端口、定位不了问题。具体场景：查接口报错的根因（看应用日志）、验证文件是否生成、确认服务进程是否存活、分析磁盘/内存是否打满导致异常。
【结合你的经验】银企直联对接 40+ 银行渠道，渠道服务异常时我第一反应是登服务器 tail 日志看银行交互报文，多次靠日志里的超时/报文错位定位到配置问题，Linux 是我排查的"第一现场"。
【追问链】不会 Linux 做测开会怎样？→ 你最常用哪几个命令？
[ ] 已掌握

### 2. 查看日志的常用命令组合
【考点】日志四板斧。
【参考答案】核心四板斧：
- `tail -f app.log` 实时跟踪日志
- `grep "ERROR" app.log` 过滤关键字
- `less app.log` 大文件分页（/ 搜索、G 末尾）
- 组合：`tail -f app.log | grep -i error` 实时看错误；`grep -C 3 "timeout" app.log` 看上下文

```bash
# 实时看某渠道报错
tail -f /data/logs/bank.log | grep -i "channel_10\|error"
# 看某时间段的错误及上下文
grep -C 5 "NullPointer" app.log | less
```
【结合你的经验】排查某银行渠道回盘延迟，我用 `tail -f` 实时跟日志，配合 `grep` 过滤渠道号，发现报文在"签名验签"节点卡住，定位到证书过期。
【追问链】日志太大怎么快速定位时间段？→ tail -f 和 tail -F 区别？
[ ] 已掌握

### 3. grep 常用参数
【考点】参数记忆。
【参考答案】
- `-i` 忽略大小写
- `-r` 递归目录
- `-n` 显示行号
- `-E` 扩展正则（支持 | + 等）
- `-A n` 显示匹配后 n 行
- `-B n` 显示匹配前 n 行
- `-C n` 前后各 n 行

```bash
grep -rin "timeout" /data/logs/        # 递归忽略大小写
grep -E "ERROR|WARN" app.log           # 多关键字
grep -A 3 -B 3 "OutOfMemory" app.log   # 上下文
```
【结合你的经验】查生产事故根因时我用 `grep -rn "渠道配置错误" /logs/` 全目录扫，配合 `-n` 定位到具体文件和行。
【追问链】grep 和 egrep 关系？→ 怎么统计匹配行数（-c）？
[ ] 已掌握

### 4. awk/sed 一句话典型用法
【考点】文本处理。
【参考答案】awk 擅长按列处理：
```bash
awk '{print $1, $4}' access.log              # 取第1、4列
awk '$9==500{print $0}' access.log           # 第9列=500的行（统计5xx）
awk '{sum+=$NF} END{print sum}' data.txt     # 末列求和
```
sed 擅长替换/删除：
```bash
sed 's/old/new/g' file.txt                   # 全局替换
sed -i 's/127.0.0.1/0.0.0.0/g' conf.yml      # 原地改配置
sed -n '10,20p' file.txt                     # 打印10-20行
```
【结合你的经验】解析银行报文日志，我用 awk 把每行第几列的交易号、金额抽出来做快速核对，比肉眼看快很多。
【追问链】awk 默认分隔符？→ sed -i 危险在哪？
[ ] 已掌握

### 5. find 查找文件
【考点】文件定位。
【参考答案】
```bash
find /data -name "*.log"                 # 按名
find /data -name "app.log" -mtime -1     # 1天内修改
find /data -size +100M                   # 大于100M
find /data -type f -mmin -30             # 30分钟内
```
【结合你的经验】磁盘告警时我用 `find /data -size +500M` 找出超大日志文件，确认是某渠道报文没轮转，清理后恢复。
【追问链】find 和 locate 区别？→ 按时间怎么组合删旧文件？
[ ] 已掌握

### 6. 查看进程：ps vs top
【考点】进程监控。
【参考答案】
- `ps` 快照：`ps -ef | grep java` 看某进程；`ps aux` 全量。
- `top` 实时：`top` 动态刷新，关键指标：load average（1/5/15分钟负载）、%CPU、%MEM、RES 常驻内存。

```bash
ps -ef | grep bank-service
top -p 1234          # 只看某进程
```
top 指标解读：load 持续 > CPU 核数说明过载；CPU 高可能是计算/GC；MEM 高可能内存泄漏。
【结合你的经验】银企直联服务响应变慢，top 一看某 Java 进程 CPU 99%，配合线程排查定位到死循环解析报文。
【追问链】load average 是什么？→ ps 的 STAT 列 R/S/D 含义？
[ ] 已掌握

### 7. 杀进程与信号
【考点】kill -9 vs -15。
【参考答案】
- `kill -15`（SIGTERM）：温和终止，进程可捕获做清理（释放资源、落盘），默认信号。
- `kill -9`（SIGKILL）：强制杀死，不可捕获，可能导致数据丢失/文件损坏，慎用。
- `kill -l` 看所有信号。

```bash
kill -15 1234     # 先礼貌终止
kill -9 1234      # 不行再强杀
```
【结合你的经验】重启渠道服务我先用 -15 让它优雅停（回滚未完成的交易），卡死才 -9，并在自动部署脚本里做了这个顺序，单渠道部署 3 分钟压到 10 秒。
【追问链】为什么不能上来就 -9？→ pkill/killall 怎么用？
[ ] 已掌握

### 8. 查看端口占用
【考点】端口排查。
【参考答案】
```bash
netstat -tunlp | grep 8080     # 传统
ss -tunlp | grep 8080          # 更快（推荐）
lsof -i:8080                   # 看占用进程
```
-t TCP / -u UDP / -n 数字 / -l 监听 / -p 进程。
【结合你的经验】FastAPI 模块部署后接口不通，ss 一看 8000 端口被旧进程占着，杀掉重启解决。
【追问链】端口被占且找不到进程？→ TIME_WAIT 占端口怎么办？
[ ] 已掌握

### 9. 磁盘与内存排查
【考点】资源打满。
【参考答案】
```bash
df -h            # 磁盘分区使用率
du -sh /data/*   # 目录大小
free -h          # 内存（含 swap）
```
排查故事：服务突然写不了文件 → `df -h` 发现 `/data` 100% → `du -sh` 定位大日志 → 清理/扩容恢复。
【结合你的经验】一次银企直联对账任务失败，查 `df -h` 根因是磁盘满，日志未轮转，清掉后任务恢复，之后我给所有渠道服务加了日志轮转。
【追问链】df 和 du 显示不一致？→ 删除文件但空间不释放为什么？
[ ] 已掌握

### 10. CPU 飙高排查思路
【考点】完整链路。
【参考答案】链路：① `top` 找出高 CPU 进程 PID；② `top -Hp PID` 看哪个线程 TID 高；③ `printf "%x\n" TID` 转十六进制；④ `jstack PID | grep -A 20 0xTID` 看线程栈定位代码；⑤ 结合应用日志确认是死循环/频繁 GC/正则灾难。
```bash
top -p 1234
top -Hp 1234
printf "%x\n" 2345
jstack 1234 | grep -A 30 0x929
```
【结合你的经验】报文比对脚本 CPU 飙高，按上面链路定位到一个"超大报文正则匹配"导致，改成分段处理后恢复正常。
【追问链】GC 导致 CPU 高怎么看？→ 没有 jstack 怎么办？
[ ] 已掌握

### 11. 文件权限 chmod/chown
【考点】权限位。
【参考答案】权限三位：所有者/组/其他，每位 r=4 w=2 x=1。
- `755`：属主 rwx(7)，组和其他 rx(5) —— 可执行程序常用
- `644`：属主 rw(6)，组和其他 r(4) —— 普通文件常用
```bash
chmod 755 deploy.sh
chmod 644 config.yml
chown app:app /data/logs   # 改属主
```
【结合你的经验】自动部署脚本必须 755 才能被执行，环境信息模块部署时我把脚本 chmod 755、日志目录 chown 给运行用户，避免权限拒绝。
【追问链】chmod +x 什么意思？→ 目录的 x 权限是什么？
[ ] 已掌握

### 12. systemctl 服务管理
【考点】服务运维。
【参考答案】
```bash
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl status nginx        # 看状态+最近日志
journalctl -u nginx -f        # 看该服务日志
systemctl enable nginx        # 开机自启
```
【结合你的经验】银企直联渠道服务用 systemd 托管，status 看存活，出问题时 `journalctl -u 渠道服务 -f` 看实时日志，比进容器方便。
【追问链】systemctl 和 service 区别？→ 怎么看服务失败原因？
[ ] 已掌握

### 13. vi/vim 常用操作
【考点】编辑器。
【参考答案】
- 查找：`/关键字` 回车，n 下一个，N 上一个
- 跳转：`gg` 首行，`G` 末行，`:100` 第100行
- 编辑：`i` 插入，`a` 行尾插，`o` 新行
- 保存退出：`:wq` 保存退，`:q!` 强退不存，`Esc` 回命令模式
- 替换：`:%s/old/new/g` 全局替换
【结合你的经验】在服务器上改渠道配置文件（如银行 IP、超时时间）时直接用 vim，`:%s` 批量改超时阈值，改完 `:wq` 重启服务。
【追问链】vim 怎么显示行号？→ 怎么整段删除（dd）？
[ ] 已掌握

### 14. curl 常用用法
【考点】接口自测。
【参考答案】
```bash
curl http://host:8000/health                 # GET
curl -X POST -H "Content-Type: application/json" \
  -d '{"channel":10}' http://host:8000/api    # POST+JSON
curl -i http://host                         # 看响应头
curl -u user:pass http://host                # 基础认证
curl -w "%{http_code} %{time_total}\n" http://host  # 状态码+耗时
```
【结合你的经验】环境信息模块联调时我用 curl 直接打 FastAPI 接口验证 JWT 鉴权（带 Bearer Token），比开 Postman 快；也用 `-w` 看响应耗时做简单压测。
【追问链】curl 怎么带 Cookie？→ 怎么忽略证书（-k）？
[ ] 已掌握

### 15. Docker 是什么？测试中怎么用？
【考点】容器概念。
【参考答案】Docker 是容器化引擎，把应用+依赖打包成镜像，运行成隔离的容器，环境一致、秒级启动。测试用法：① 一键拉起被测环境（MySQL/Redis/被测服务）；② 多版本并行（同时跑 MySQL5.7 和 8.0）；③ CI 里构建镜像跑测试；④ 本地复现生产环境排错。
【结合你的经验】我独立开发的环境信息模块就容器化部署，Docker 保证测试/生产环境一致，避免"我本地能跑"。回归时也用 Docker 起临时 Oracle/MySQL 造数。
【追问链】容器和虚拟机区别？→ 镜像分层有什么好处？
[ ] 已掌握

### 16. Docker 常用命令 20 个
【考点】命令清单。
【参考答案】
```bash
docker --version            # 版本
docker images               # 镜像列表
docker ps -a                # 容器列表(含停止)
docker pull mysql:8        # 拉镜像
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=1 mysql:8  # 起容器
docker build -t myapp .    # 构建
docker start/stop/restart id # 启停
docker exec -it id bash     # 进容器
docker logs -f id           # 看日志
docker cp a.txt id:/data/   # 拷文件
docker rm id / docker rmi id # 删容器/镜像
docker inspect id           # 详情
docker network ls           # 网络
docker volume ls            # 数据卷
docker compose up -d       # 编排启动
docker system prune         # 清理
```
【结合你的经验】部署 FastAPI 模块我用 `docker build` + `docker run -d -p` 起服务，`docker logs -f` 看启动日志，`docker exec` 进容器查配置，整套流程烂熟。
【追问链】-d 和 -it 区别？→ 数据怎么持久化（volume）？
[ ] 已掌握

### 17. Dockerfile 基本结构与常用指令
【考点】手写 Dockerfile（配 FastAPI 示例）。
【参考答案】常用指令：FROM（基础镜像）、WORKDIR（工作目录）、COPY（拷文件）、RUN（构建期命令）、EXPOSE（声明端口）、ENV（环境变量）、CMD/ENTRYPOINT（启动命令）。
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
COPY . .
ENV ENV=prod
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```
【结合你的经验】环境信息模块就是这个 Dockerfile 部署的，用 python:3.11-slim 减小镜像，多阶段构建进一步优化，单渠道部署从 3 分钟降到 10 秒正是靠容器化+自动部署。
【追问链】CMD 和 ENTRYPOINT 区别？→ 多阶段构建怎么写？
[ ] 已掌握

### 18. 镜像 vs 容器 vs 仓库
【考点】核心概念。
【参考答案】镜像（Image）是只读模板（类"类"）；容器（Container）是镜像运行实例（类"对象"），可读写层；仓库（Registry）是存镜像的地方（Docker Hub / 私有 Harbor）。关系：从仓库 pull 镜像 → run 成容器 → 改了可 commit 成新镜像 push 回仓库。
【结合你的经验】我们的 FastAPI 镜像推到公司私有仓库，CI 自动构建，测试环境 pull 下来跑回归，保证版本一致。
【追问链】镜像和容器的存储层？→ 仓库和注册表区别？
[ ] 已掌握

### 19. 容器日志怎么看？
【考点】docker logs。
【参考答案】
```bash
docker logs -f id          # 实时
docker logs --tail 100 id # 最后100行
docker logs --since 30m id # 近30分
docker logs --until 2026-01-01 id
```
容器日志默认 json-file 驱动，存在 /var/lib/docker，生产需配日志轮转（不然打满磁盘）。
【结合你的经验】k8s 之前我们在单机 Docker 跑渠道服务，出问题时 `docker logs -f --tail 200` 看最近报错，配合 grep 找银行报文异常。
【追问链】日志驱动有哪些？→ 容器日志太大怎么限大小？
[ ] 已掌握

### 20. k8s 是什么？测试怎么用 k8s 排查？
【考点】容器编排 + 排查。
【参考答案】k8s 是容器编排平台，管理多容器部署、扩缩容、自愈。测试排查四板斧：
```bash
kubectl get pods -n env          # 看 Pod 状态(Running/CrashLoop)
kubectl describe pod xxx -n env  # 看事件(拉镜像失败/探针失败)
kubectl logs -f xxx -n env        # 看日志
kubectl exec -it xxx -n env -- bash  # 进容器
```
【结合你的经验】银企直联上了 k8s 后，渠道异常我先 `kubectl get pods` 看是否 CrashLoopBackOff，再 `describe` 看 Events 发现是配置 Map 挂载失败，最后 `logs` 确认银行证书读取异常——这套链路我每周都用。
【追问链】Pod 状态有哪些？→ 探针 liveness/readiness 区别？
[ ] 已掌握

### 21. TCP 三次握手
【考点】建连过程 + 为什么不是两次。
【参考答案】① 客户端发 SYN（seq=x）；② 服务端回 SYN+ACK（seq=y, ack=x+1）；③ 客户端发 ACK（ack=y+1），连接建立。为什么不是两次：若只有两次，服务端无法确认客户端的接收能力，且旧的滞后 SYN 可能让服务端误建连接浪费资源；三次握手让双方都确认"能发能收"。
【结合你的经验】压测银行渠道接口时偶发连接失败，抓包看是三次握手第 2 步 SYN-ACK 丢，根因是服务端半连接队列满（SYN Flood 防护过严），调大后恢复。
【追问链】半连接队列是什么？→ 为什么是三次不是四次？
[ ] 已掌握

### 22. TCP 四次挥手
【考点】断连 + TIME_WAIT。
【参考答案】① 客户端 FIN（关闭发送）；② 服务端 ACK；③ 服务端 FIN（数据发完才发）；④ 客户端 ACK。TIME_WAIT：主动关闭方发完最后 ACK 后等 2MSL，确保对方收到、让旧报文消散。过多 TIME_WAIT 会占端口。
【结合你的经验】高并发短连接压测后出现"端口耗尽"，`ss -tan` 看到大量 TIME_WAIT，靠开启端口复用（reuse/tw_reuse）+ 用长连接解决。
【追问链】为什么挥手要四次？→ 2MSL 是多久？
[ ] 已掌握

### 23. TCP vs UDP 区别
【考点】特性与场景。
【参考答案】TCP 面向连接、可靠（确认/重传/有序）、慢，适合数据不能丢（HTTP/文件/数据库）；UDP 无连接、不可靠但快、可广播，适合实时（音视频/游戏/DNS）。测试意义：UDP 丢包不重传，压测时要看丢包率；TCP 看吞吐和时延。
【结合你的经验】银企直联有走 TCP 长连接的渠道（可靠），也有 UDP 报文通道，测 UDP 渠道时我重点关注丢包和乱序，用 Fiddler/抓包核对。
【追问链】TCP 怎么保证可靠？→ 哪些应用层用 UDP？
[ ] 已掌握

### 24. HTTP 常见状态码
【考点】每个配测试场景。
【参考答案】
- 200 成功（接口正常）
- 301 永久重定向 / 302 临时重定向（登录跳转）
- 400 请求参数错（入参校验失败）
- 401 未认证（Token 过期/缺失）
- 403 禁止（无权限，RBAC 拦截）
- 404 资源不存在（路径错/版本下线）
- 500 服务器内部错（代码异常，重点查日志）
- 502 网关错（后端挂了/Nginx 转发失败）
- 503 服务不可用（过载/维护）
- 504 网关超时（后端响应慢，常见慢 SQL/外部银行超时）

```bash
curl -i http://host/api | head -1   # 看状态码
```
【结合你的经验】银企直联报错 504，根因是银行侧响应超 30s 触发网关超时，我调大超时阈值并重试机制；401 则是 JWT 过期，前端刷新 Token。
【追问链】401 和 403 区别？→ 502 和 504 区别？
[ ] 已掌握

### 25. GET vs POST 区别
【考点】语义与差异。
【参考答案】GET 取数据，参数在 URL、有长度限制、可缓存、幂等、不安全；POST 提交数据，参数在 body、无长度限制、非幂等（可能改状态）、相对安全。本质上两者都是 TCP，技术上 GET 也能带 body，但语义和规范约定不同。RESTful 里 GET 查、POST 增。
【结合你的经验】资金系统接口严格区分：查询交易用 GET，发起扣款/对账用 POST，避免幂等问题（GET 被重试不会重复扣款）。
【追问链】GET 真的不能带 body 吗？→ 幂等是什么意思？
[ ] 已掌握

### 26. HTTP vs HTTPS
【考点】加密过程。
【参考答案】HTTPS = HTTP + TLS/SSL。过程：① 客户端请求，服务端发证书（含公钥）；② 客户端验证证书（CA 链）；③ 协商对称密钥（用公钥加密随机数发给服务端，或用 DH 算法）；④ 之后用对称密钥加密通信。非对称加密用于"安全交换密钥"，对称加密用于"高效传数据"。
【结合你的经验】银企直联对银行侧强制 HTTPS+双向证书（mTLS），我测联调时验证过证书校验逻辑——证书错/过期直接 401，抓包看是密文。
【追问链】为什么不全用非对称？→ 对称加密密钥怎么来的？
[ ] 已掌握

### 27. Cookie vs Session vs Token
【考点】认证三件套。
【参考答案】
- Cookie：存在浏览器的键值，随请求自动带，大小受限，可设 HttpOnly/Secure。
- Session：存在服务端的状态（如用户登录信息），靠 Cookie 里的 sessionId 关联。
- Token：无状态令牌（如 JWT），客户端存，每次请求带，服务端不存状态，适合分布式/前后端分离。
【结合你的经验】环境信息模块用 Token（JWT）而非 Session，因为前后端分离+多服务，Session 共享麻烦；Token 无状态便于 k8s 多副本横向扩展。
【追问链】Session 共享怎么解决？→ Token 被盗怎么办？
[ ] 已掌握

### 28. JWT 结构
【考点】Header.Payload.Signature（结合实践）。
【参考答案】JWT 三段 base64：
- Header：`{alg:HS256, typ:JWT}`
- Payload：存 claims（用户ID、角色、过期 exp）
- Signature：HMAC(Header.Payload, 密钥) 防篡改
校验时服务端用密钥重算签名比对，不查库即可认证，但无法主动注销（靠 exp 过期）。
```text
eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiWXMiLCJyb2xlIjoiYWRtaW4iLCJleHAiOjE5fQ.xxxxxx
```
【结合你的经验】我开发环境信息模块时自写 JWT 鉴权：登录发 Token（含 user_id+RBAC 角色），FastAPI 用依赖注入校验签名+exp，拦截越权；白盒测试时专门测了"改 Payload 角色"被签名校验拒绝。
【追问链】JWT 怎么主动退出？→ 密钥泄露风险？
[ ] 已掌握

### 29. RESTful API 设计规范
【考点】接口风格 + 资金例子。
【参考答案】核心：用资源 URL + HTTP 方法表达操作；无状态；返回标准状态码+JSON。
- GET /trans/{id} 查交易
- POST /trans 创建交易
- PUT /trans/{id} 全量改
- DELETE /trans/{id} 删
- 版本：`/api/v1/...`；过滤 `?status=S&page=1`
【结合你的经验】环境信息模块接口严格 REST 风格：GET 查环境信息、POST 新增、JWT 鉴权，Swagger 自动文档；资金系统对外也用 `/api/v1/channels/{id}/trans` 这类资源式路径。
【追问链】REST 和 RPC 区别？→ 怎么设计分页/过滤？
[ ] 已掌握

### 30. Fiddler 抓包 + DNS/OSI
【考点】抓包实战 + 网络模型。
【参考答案】Fiddler 用法：
- 弱网模拟：Rules → Performance → Simulate Modem Speeds，或自定义上传/下载延迟，测超时与重试。
- 断点：在请求前/响应后设断点（`bpu` / `bpafter`），改参数/响应，测前端容错。
- Mock 响应：AutoResponder 返回伪造 JSON，模拟银行回盘/异常码，隔离外部依赖测前端。
- DNS 解析：浏览器输域名 → 查本地 hosts/Cache → 递归查根/顶级/权威 DNS → 拿 IP → 建 TCP。
- OSI 七层（一句话）：物理→数据链路→网络(IP)→传输(TCP/UDP)→会话→表示→应用(HTTP)，下三层管"通"，上四层管"懂"。
【结合你的经验】测银企直联前端时，我用 Fiddler 的 AutoResponder Mock 银行"退票/超时"响应，验证前端提示；弱网模拟测出过"超时未提示重试"的 bug；断点改报文测越权。
【追问链】Fiddler 和 Charles 区别？→ 七层各举一个协议？
[ ] 已掌握

---

## 二、必背清单

**日志四板斧**：tail -f / grep / less / 组合管道
**grep**：-i 忽略大小写 -r 递归 -n 行号 -E 正则 -A/-B/-C 上下文
**awk**：按列处理 `{print $1}`、`$9==500` 过滤、`END` 汇总
**sed**：`s/old/new/g` 替换、`-i` 原地、`-n '10,20p'` 取行
**find**：-name / -mtime / -size / -type
**进程**：ps 快照 / top 实时（load、cpu、mem）
**kill**：-15 优雅 / -9 强杀（慎用）
**端口**：netstat -tunlp / ss -tunlp / lsof -i:port
**磁盘内存**：df -h / du -sh / free -h
**CPU飙高**：top→top -Hp→printf %x→jstack 定位线程
**权限**：755=rwxr-xr-x / 644=rw-r--r--；chmod/chown
**systemctl**：start/stop/status/enable；journalctl -u 看日志
**vim**：/查找、gg/G跳转、i编辑、:wq保存、:%s替换
**curl**：-X POST -H -d -i -u -w（状态码耗时）
**Docker**：images/ps/run/build/exec/logs/cp/rm；镜像=模板 容器=实例 仓库=存镜像
**Dockerfile**：FROM/WORKDIR/COPY/RUN/EXPOSE/ENV/CMD
**k8s**：get pods / describe / logs / exec（排查四板斧）
**TCP**：三次握手（确认双发收） / 四次挥手 / TIME_WAIT 等2MSL
**TCP vs UDP**：可靠有序 vs 快不可靠
**HTTP码**：200/301/302/400/401/403/404/500/502/503/504（504=后端超时）
**GET vs POST**：URL参数幂等 vs body非幂等
**HTTPS**：非对称交换密钥 + 对称传数据
**认证**：Cookie(浏览器)/Session(服务端)/Token(无状态JWT)
**JWT**：Header.Payload.Signature，签名防篡改，靠 exp 过期
**REST**：资源URL+方法，无状态，版本/api/v1
**Fiddler**：弱网/断点/Mock；DNS 递归解析；OSI 七层下三通上四懂

---

## 三、实战演练

**演练1：服务 500 报错，怎么一步步定位？**
1. `kubectl get pods` 看是否 CrashLoop / `docker ps` 看状态
2. `kubectl logs -f pod` / `docker logs -f` 看异常栈（NullPointer/SQL 错）
3. `tail -f app.log | grep ERROR` 看业务日志
4. 若有 SQL 错 → EXPLAIN 查慢查询；若 OOM → `top`/`free -h` 看内存
5. 定位后改配置/代码，重启验证

**演练2：接口响应慢（>3s）**
1. `curl -w "%{time_total}"` 量化耗时
2. 看是不是 504（网关超时=后端慢）
3. 进容器/服务器 `top` 看 CPU，EXPLAIN 看 SQL
4. 银企直联场景优先怀疑"银行侧响应慢"——看日志里报文交互耗时

**演练3：银行渠道连不上**
1. `ss -tunlp | grep 端口` 看本端是否监听
2. `ping`/`telnet host port` 看网络通不通
3. `tcpdump` 或 Fiddler 抓包看三次握手是否完成
4. 看证书（HTTPS/mTLS）是否过期 → 日志里 TLS 错误

**演练4：用 Docker 起一套回归环境**
```bash
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=1 -v ./init.sql:/docker-entrypoint-initdb.d/init.sql mysql:8
docker build -t bank-test .
docker run -d -p 8000:8000 --link mysql bank-test
curl http://localhost:8000/health
```

**演练5：Fiddler Mock 银行回盘（断点改响应）**
1. 设 `bpafter api/bank/callback` 拦截回盘响应
2. 在响应里把 status 改成 "REJECTED"、金额改 0
3. 放行，验证前端是否提示"退票"且对账标记失败
4. 这一招能在不依赖真实银行的情况下，把异常分支全覆盖
