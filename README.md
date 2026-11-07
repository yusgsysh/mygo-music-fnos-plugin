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
- 专辑/艺术家/歌曲封面同样需要会话 Cookie。MyGo Music 从上游 commit [`b192406`](https://github.com/egoist/mygo-music/commit/b19240642f0cc08ddad12376b29afe513d5eda96) 起支持插件的 `imageHeaders({url})`；插件会为飞牛服务器的封面返回带 `music-token` 的 `Cookie`，因此封面可以正常显示。旧版 MyGo Music 会忽略这个导出，封面显示为占位图，播放、搜索、歌词和歌单不受影响。

## 封面

插件的 `imageHeaders` 只对飞牛服务器的 `/music/api/v1/static/cover` 地址返回：

```js
{ headers: { Cookie: "music-token=..." }, maxAge: 600 }
```

封面 URL 本身不包含 token，所以保存的歌曲不会留下会话凭证；token 更新后，下一次询问就会使用新的值。MyGo Music 会把请求头发送给该封面域名及其子域，并在收到 401/403 时重新询问插件。

## 开发

插件就是一个 `plugin.json` 加一个 `index.js`，没有构建步骤。接口形状见 MyGo Music 的 `internal/music/music.go`。
