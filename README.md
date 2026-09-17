# 手摇风琴纸带打孔API

纯后端零依赖Node服务，使用 `data/db.json` 持久化曲目、纸带区间和试奏问题。

## 启动

```bash
PORT=3019 node server.js
```

## 主要接口

- `GET /health`
- `GET /tunes`
- `POST /tunes`
- `GET /tunes/:id/progress`
- `GET /tunes/:id/sections`
- `POST /tunes/:id/sections`
- `GET /tunes/:id/unchecked-sections`
- `PATCH /sections/:id/check`
- `GET /issues?tuneId=&status=`
- `POST /issues`
- `PATCH /issues/:id/status`

## 闭环示例

```bash
curl http://127.0.0.1:3019/tunes/tune_demo/progress
curl -X POST http://127.0.0.1:3019/issues \
  -H 'Content-Type: application/json' \
  -d '{"tuneId":"tune_demo","sectionId":"section_demo_2","type":"错孔","beat":45,"lane":9,"description":"第45拍第9轨多打孔"}'
```

## 区间与问题的状态联动

区间的 `checked` 与问题的 `status` 保持双向一致（未解决 = `status !== "resolved"`）：

- `PATCH /sections/:id/check` 把区间由未校对改为已校对时，若仍有未解决问题，返回 **409**，区间保持 `unchecked`，不写入任何改动，问题状态也不变；响应体附 `openIssueCount` 与 `openIssueIds`。对已经是 `checked` 的区间重复校对视为幂等操作。
- 手工把区间改回 `unchecked`（`{"checked": false}`）只改区间本身，**不会**重开任何已解决问题。
- `PATCH /issues/:id/status` 关闭一个问题后，若它是该区间最后一个未解决问题，区间自动置为 `checked`。
- 重新打开任一已解决问题（如 `open`/`reopened`），区间自动改回 `unchecked`。
- `POST /issues` 给区间登记新问题时，该区间自动改回 `unchecked`。
- 更新在服务端串行化（互斥锁 + 临时文件原子落盘），并发 PATCH 不会互相覆盖。
- 所有既有响应字段与外层 `{ data }` / `{ error }` 结构保持兼容。
