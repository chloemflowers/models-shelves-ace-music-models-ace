# 模型上架：ACE 音乐模型（ace-step-1.5-xl、acemusic-v1、acemusic-v1.1）

2026 年 11 月 21 日凌晨（系统公告发布时间约 02:33，Asia/Taipei，UTC+8），第三方接口聚合站 momoAPIPro（[tryallapi.com](https://tryallapi.com/)）在系统公告里上新了一组音乐模型：

> 通知：【suno-上新】ACE 音乐模型上线！解决 Suno 翻唱版权拦截，翻唱效果对标 Suno，支持 ace-step-1.5-xl /acemusic-v1 /acemusic-v1.1，提供灵感、歌词、翻唱三种模式。

![tryallapi 系统公告：ACE 音乐模型上线](https://upload.maynor1024.live/file/1795223323885_01-announcement.jpg)

> 图注：2026-11-21 站点系统公告截图。上方是 ACE 音乐模型上线通知；下方被截断的一条是 Qwen4-Max 缓存计价通知，与本文无关。

要在自己的项目或脚本里按次调用，可以从这个注册入口进入：[https://tryallapi.com/register?aff=5A6A](https://tryallapi.com/register?aff=5A6A)。

本文按 **2026 年 11 月 21 日（Asia/Taipei，UTC+8）** 对系统公告、站点接口文档、公开定价接口，以及 ACE-Step 官方资料的复核来写。先把几件事说清楚：

- tryallapi.com / momoAPIPro 是**第三方 API 聚合站**，不是 Suno 官方，也不是 ACE Studio、阶跃星辰（StepFun）或 ACE Music（acemusic.ai）的官方 API。
- 公告前面的「【suno-上新】」是站点的**分类标签**：ACE 是通过站点 Suno 接口调用的另一组音乐模型，**不是 Suno 发布的模型**。
- 「解决 Suno 翻唱版权拦截」「翻唱效果对标 Suno」都是**公告称**，本文没有做效果评测，也不把它当成已验证的结论。
- 模型是否在线、分组是否开放、价格都会变，请以当天控制台、模型广场和文档为准。

## 三个模型 ID 分别是什么

站点文档《创建音乐》（`POST /suno/submit/music`）的 `mv` 参数里，在原有 chirp 系列之后新增了三个 ACE 版本，文档原样写的是：

| `mv` 填写值 | 文档里的对应名称 |
| --- | --- |
| `acemusic-v1.1` | ACE Music v1.1 Preview（文档标注「推荐」） |
| `acemusic-v1` | ACE Music v1 |
| `ace-step-1.5-xl` | ACE-Step v1.5 XL |

对照官方资料，可以核实到的背景是：

- **ACE-Step** 是由 ACE Studio 与 StepFun（阶跃星辰）共同主导的开源音乐生成项目（[GitHub：ace-step/ACE-Step-1.5](https://github.com/ace-step/ACE-Step-1.5)）。官方仓库记载，**ACE-Step 1.5 XL** 于 2026 年 4 月 2 日发布，采用 4B 参数的 DiT 解码器，有 xl-base、xl-sft、xl-turbo 三个变体；权重公开在 [Hugging Face 的 ACE-Step 组织](https://huggingface.co/ACE-Step) 下（如 `acestep-v15-xl-base`、`acestep-v15-xl-sft`、`acestep-v15-xl-turbo`）。项目页见 [ACE-Step 1.5](https://ace-step.github.io/ace-step-v1.5.github.io/)。
- **ACE Music v1 / ACE Music v1.1 Preview** 是 ACE Music 网站（[acemusic.ai](https://acemusic.ai/)，官方仓库首页有推荐链接）创作页里的模型名。按其帮助文档《[Choose an ACE Music model](https://acemusic.ai/help/articles/choose-an-ace-music-model/)》，当前模型选择器列出 ACE Music v1、ACE Music v1.1 Preview 和 ACE-Step v1.5 XL：v1 偏向按描述或歌词创作；v1.1 Preview 用于旋律 / 乐谱参考和 Mashup；ACE-Step v1.5 XL 支持文本生成，以及单曲 Cover / Edit。

需要注意：`acemusic-v1`、`acemusic-v1.1`、`ace-step-1.5-xl` 这三个**写法是 tryallapi 文档里的参数值**，它与上面官方名称的对应关系来自 tryallapi 文档本身。本文无法核实 tryallapi 背后具体接的是哪个上游、哪个变体（例如 XL 的 base / sft / turbo），也不代表 ACE Music 或 ACE-Step 官方提供或背书这条通道。acemusic.ai 页面上的 Credits、免费额度、下载格式、商用条款，只适用于 acemusic.ai 本身，**不要套到 tryallapi 上**。

## 怎么调用（按站点文档）

站点文档写明，ACE 不是单独的新接口，而是复用 Suno 的「创建音乐」接口：

1. 接口：`POST /suno/submit/music`，提交后按异步任务用 `POST /suno/fetch` 查询结果。Base 地址以控制台和 [文档](https://docs.tryallapi.com/reference/suno/submit/music) 为准。
2. 切换方式：`mv` 填 ACE 版本号即切到 ACE 音乐模型。文档说网关会自动带上上游参数 `expect_ai_model=ace`，通常不用自己填；其余参数与 Suno 相同。
3. 文档写明 ACE 支持**灵感模式、自定义模式、翻唱**。公告里说的「灵感、歌词、翻唱」三种模式，可以这样对照理解：
   - 灵感（描述）模式：填 `gpt_description_prompt`，由模型生成风格与歌词；
   - 歌词（自定义）模式：填 `prompt`（歌词）+ `tags`（风格）；ACE 版本还支持可选的 `negative_tags`（不想要的风格）；
   - 翻唱：`cover_clip_id` 可以直接填音频 URL（文档推荐），或先用 `POST /suno/uploads/audio` 上传参考音频（此时要带 `expect_ai_model: "ace"`，因为上传接口没有 `mv`），拿到 `so_` 开头的 ID 再填进来。
4. 文档里的最简请求示例：

```json
{
  "gpt_description_prompt": "一首轻快的夏日流行歌",
  "make_instrumental": false,
  "mv": "acemusic-v1.1"
}
```

还有两点容易混淆：

- **这里的「灵感模式」和之前上架的「Suno 灵感模式（MIDI 旋律续写）」不是一回事。** 文档写明 MIDI 创作是 Suno 专属，模型固定为 `chirp-v6-wild`，**不能和 ACE 版本一起用，会返回 400**。ACE 的灵感模式只是按文字描述生成歌曲。
- 公告写的三种模式与文档的字段写法不完全一一对应，例如歌词与风格的字段同时填写时，文档提示以网关 / 上游实际分支为准。正式接入前，最好先用少量请求在控制台确认每种模式的返回。

## 价格和分组

- 截至 2026-11-21，公开定价目录里**没有**单独的 `ace-step-1.5-xl`、`acemusic-v1`、`acemusic-v1.1` 条目。
- 文档写明，ACE 走的是同一个「创建音乐」接口，在模型广场和消费记录里显示为 **`suno_music_open`**，并写明「**价格与 Suno 相同**」。
- 当天目录里 `suno_music_open` 是**按次计费**（不是按 token），可用分组是 **Suno-1**（倍率约 x0.1618）和 **Suno-2**（倍率约 x0.2427）。这两个是 tryallapi 的分组名，不是 Suno 官方的产品线。
- 不同操作（发起创作、上传参考音频等）的按次单价，请在模型广场打开 `suno_music_open` 详情核对，本文不另编一张 ACE 专属价目表，也不推算「每首歌多少钱」「每秒多少钱」。实际扣费以控制台账单和当时命中的分组为准。

上架不等于每个分组一直可用，也不承诺生成时长、成功率或音质。

## 关于翻唱和版权

公告称 ACE「解决 Suno 翻唱版权拦截」，文档也写着翻唱流行歌曲 / 已在 Suno 库里的歌「不会被 Suno 版权拦截」。这只说明**这条接口不会触发 Suno 那一侧的拦截**，**不等于你获得了这首歌的授权**：

- 翻唱、改编他人受版权保护的歌曲，发布或商用时通常仍需要相应的授权（词曲、录音等权利各不相同）。
- **能生成 ≠ 可以商用或公开发布。** 上传的参考音频请确保你有权使用。
- 本文不提供法律意见；涉及发布、商用、平台投稿时，请按所在地区法规和各平台规则自行确认，必要时咨询专业人士。

## 快速上手

1. 通过 [https://tryallapi.com/register?aff=5A6A](https://tryallapi.com/register?aff=5A6A) 注册并登录控制台。
2. 新建令牌时，确认分组能用到 Suno-1 或 Suno-2（以当天控制台为准）。
3. 调用 `POST /suno/submit/music`，把 `mv` 改成 `acemusic-v1.1`、`acemusic-v1` 或 `ace-step-1.5-xl`。
4. 用 `POST /suno/fetch` 查询任务状态并取回音频。
5. 先小批量试跑三种模式，对照消费记录里 `suno_music_open` 的实际扣费，再决定要不要批量使用。

## 需要记住的几点

- tryallapi / momoAPIPro 是第三方按次通道，不是 Suno、ACE Studio、StepFun 或 ACE Music 的官方服务。
- 「suno-上新」是站点分类标签；ACE 不是 Suno 的模型。
- 三个模型 ID 是 tryallapi 文档里 `mv` 的取值；ACE-Step 1.5 XL 的开源背景可在官方 GitHub / Hugging Face 核实，具体上游变体无法核实。
- 「解决翻唱版权拦截」「效果对标 Suno」是公告说法；能生成不等于有版权授权。
- ACE 的灵感模式不是 MIDI 创作；MIDI 创作只支持 Suno 的 `chirp-v6-wild`。
- 计费按文档与 Suno 相同、记在 `suno_music_open` 下；价格、分组和可用性都会变，请以 2026 年 11 月 21 日之后的当天页面为准。
