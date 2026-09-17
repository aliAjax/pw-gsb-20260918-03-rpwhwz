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

## 区间与问题的状态约束

- 区间还有未解决问题（`status != "resolved"`）时，`PATCH /sections/:id/check` 标记已校对返回 `409`，区间保持 `checked: false`。
- 关闭区间最后一个未解决问题时，区间自动变为 `checked: true`。
- 重新打开任一已解决问题（状态改为非 `resolved`）时，区间自动变回 `checked: false`；在已校对区间新建问题同样会使其失效。
- 手工取消校对（`checked: false`）只改区间状态，不会重开任何已解决问题。
- 所有写操作串行执行且落盘为原子写，并发更新不会互相覆盖。

## 闭环示例

```bash
curl http://127.0.0.1:3019/tunes/tune_demo/progress
curl -X POST http://127.0.0.1:3019/issues \
  -H 'Content-Type: application/json' \
  -d '{"tuneId":"tune_demo","sectionId":"section_demo_2","type":"错孔","beat":45,"lane":9,"description":"第45拍第9轨多打孔"}'
```
