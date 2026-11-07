# 飞牛音乐 (fnOS Music) for MyGo Music

一个 [MyGo Music](https://github.com/egoist/mygo-music) 插件，把飞牛 OS 的「飞牛音乐」接到播放器里。

## 功能

- 首页：最近添加、专辑、艺术家、播放列表
- 歌曲、专辑、艺术家、播放列表的完整浏览
- 搜索歌曲、专辑、艺术家、播放列表，并带搜索建议
- 喜欢/取消喜欢歌曲（与服务器同步）
- 最近播放
- 歌词（LRC）
- 直接播放服务器上的原始音频（MP3、FLAC、WAV、OGG、M4A、AAC）
- 新建、重命名、删除播放列表，添加/移除歌曲

## 安装

### 从 GitHub 安装

仓库：https://github.com/yusgsysh/mygo-music-fnos-plugin

在 MyGo Music 的「设置 → 插件 → +」中填入：

```
yusgsysh/mygo-music-fnos-plugin
```

或直接打开安装链接：

```
mygo-music://install-plugin?repo=yusgsysh/mygo-music-fnos-plugin
```

### 从本机安装

把本目录放进 MyGo Music 的插件目录：

```
Settings > Plugins > + > Show Plugins Folder
```

然后在插件设置里填写：

| 设置 | 说明 |
| --- | --- |
| Server | 飞牛音乐地址，例如 `https://fnos.example.com`。不要带 `/music/login`。 |
| Username | 飞牛音乐用户名 |
| Password | 飞牛音乐密码 |

填好后在插件页面点登录即可。插件会在第一次用到时自动登录，会话过期后也会自动重新登录。

## 说明

飞牛音乐的 API 位于 `/music/api/v1`。它的每个请求都要带一个 `authx` 签名头，登录后的会话保存在 `music-token` Cookie 里；这个插件把两者都按网页版的方式实现了，所以不需要额外安装任何东西。

- 密码会用 SHA-256 处理后发送，和网页版一致。
- 播放地址是服务器上的原始文件地址，播放时带上会话 Cookie。
- 专辑/艺术家/歌曲封面同样需要会话 Cookie。插件会在封面地址的 fragment 里带上 `#mygo-cookie=...`，并由附带的小补丁让 MyGo Music 把它转成 `Cookie` 请求头（同时从 URL 中移除）。应用补丁并重新构建 MyGo Music 后，封面即可正常显示。补丁只改封面下载函数，不影响播放或数据；不应用补丁也能使用，只是封面会显示为占位图。

## 封面支持（可选补丁）

插件本身无法给 MyGo Music 的封面请求加请求头，因此附带了一个很小的 MyGo Music 补丁（基于 0.1.18 / main）：

```sh
cd mygo-music
git apply /path/to/patches/mygo-cover-cookie.patch
go test ./internal/music
go test ./internal/app -run TestSizedKeepsCoverCookie
go tool mygo build
```

补丁内容：

- `internal/music/fetch.go`：读取封面 URL fragment 上的 `mygo-cookie`，作为 `Cookie` 请求头发送，并从实际请求 URL 中删除。
- `internal/music/fetch_test.go`：验证 Cookie 被正确转换，且 fragment 不会发送给服务器。
- `internal/app/artwork_test.go`：验证 fragment 在 `sized` 追加 `#<px>` 后仍然保留。

应用后，插件返回的封面地址形如：

```
https://fnos.example.com/music/api/v1/static/cover?coverId=...#mygo-cookie=music-token%3D...
```

MyGo Music 会把 `mygo-cookie` 转成 `Cookie: music-token=...` 后再请求封面。fragment 不会发送给服务器，所以即使没有应用补丁，也不会把会话 token 暴露在服务器日志或查询参数里。

上游提交：[egoist/mygo-music#28](https://github.com/egoist/mygo-music/issues/28)。该仓库目前设置为只允许协作者创建 PR，因此先以 issue + patch 的形式提交；如果之后开放 PR，可直接使用 `yusgsysh/mygo-music` 的 `cover-url-cookie` 分支。

## 开发

插件就是一个 `plugin.json` 加一个 `index.js`，没有构建步骤。接口形状见 MyGo Music 的 `internal/music/music.go`。
