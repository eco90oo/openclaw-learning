# DevOps 學習記錄

## 進度追蹤

- [x] Day 1: Docker 基礎與容器化概念 (2026-03-28)
- [x] Day 2: Docker 指令詳解（run, build, ps, exec） (2026-03-29)

---

## 學習重點

### Day 1: Docker 基礎與容器化概念
- Docker 三大核心：Image（映像檔）、Container（容器）、Registry（倉庫）
- Docker 架構：Client-Server 模式
- 安裝流程與驗證
- 重要術語：Namespace、Cgroup、UnionFS

### Day 2: Docker 指令詳解
- `docker run`：啟動容器，詳解所有選項（-d, -p, -e, -it, -v, --name, --rm）
- `docker ps`：查看容器列表，輸出格式化
- `docker exec`：進入容器執行命令，互動模式
- `docker build`：從 Dockerfile 建構映像檔
- 其他指令：stop, rm, logs, stats, inspect, cp, tag, system prune

---

## 待完成

- [ ] Day 3: Dockerfile 語法詳解
- [ ] Day 4: Docker Compose 多容器部署
- [ ] Day 5: Docker 網路與Volume
- [ ] Day 6: Docker 優化（多階段建構、.dockerignore）
- [ ] Day 7: Docker 安全最佳實踐
- [ ] Day 8: Kubernetes 基礎概念
- ...（共24天）