# 小狼毫 PR #1939 · 系统应用输入测试笔记

日期：2026-10-02 20:00 前后 · 被测机器：HYPXTMC-LYH（Win10 19045，session 1）

## 起因

上游 rime/weasel PR **#1939**「feat(WeaselUI): support background image for candidate window」，
2026-10-01 由 hypxtmc 提交，当前 Open。

**维护者 fxliang 于 2026-10-02 16:19 回复一句：**

> 要测试系统应用上输入，比如开始菜单

## 今天测出来的事实（按证据强度）

### 1. 小狼毫确实注册着、且跑的是补丁版 —— 已证实

- TSF TIP CLSID **`{A3F4CDED-B1E9-41EE-9CA6-7B4D0DE6CB0A}`** 的 `InprocServer32`
  → **`C:\WINDOWS\system32\weasel.dll`**（1,074,176 字节，2026-10-01 22:01:10）
- 与 `C:\Program Files\Rime\weasel-0.17.4\weaselx64.dll` **同尺寸、同时间戳** ⇒ 注册的那份就是补丁版
- 博士的 Win+Space 切换器截图里，「中文(简体，中国) 小狼毫」处于选中态
- ⚠️ 教训：我曾把上面的 GUID 凭印象认成「微软五笔」，**认 GUID 必须查 `InprocServer32`，不许凭印象**

### 2. 配置链路完整 —— 已证实

- `%APPDATA%\Rime\weasel.custom.yaml` 有 `style/background_image: skins/endfield-light.png`
- `%APPDATA%\Rime\build\weasel.yaml` **第 545 行**同项存在（build 时间 2026-10-02 01:28）⇒ 部署过
- `skins/endfield-light.png` 3037 字节，仍在

### 3. 普通应用（记事本）里渲染正常 —— 已证实

截图里候选窗 `1 你好 2 利好 3 理好 …`：

| 观测量 | 截图所见 | 他的 endfield 配色值 | 判定 |
|---|---|---|---|
| 高亮底 | 亮黄 | `hilited_candidate_back_color: 0xFFFBFB45` | ✅ 对上 |
| 序号色 | 青 | `label_color: 0xFF2E9CB8` | ✅ 对上 |
| 候选窗底 | 近白 | `back_color: 0xFFF6F7F9` | ✅ 对上 |
| 边框 | 细灰 | `border_color: 0xFFBFC7D2` | ✅ 对上 |

⇒ 这是**小狼毫自己的面板**，不是系统 IME 的。**无背景图系博士本人要求改成纯配色**，非缺陷。

### 4. 系统应用（开始菜单）—— **未测成，不可下任何结论**

两次尝试都没把开始菜单打开（点任务栏坐标偏了，反把 MuMu 模拟器窗口拽到前台）。
**不许向 PR 提交任何关于系统应用的结论。**

## 三条踩过的坑

1. **Windows 输入法是按窗口/线程记的** —— 在 QQ 里按 Win+Space 切不到记事本。
   测某个应用的 IME，必须**先聚焦那个窗口**再切、再打字，否则测的是别的窗口。
2. **桥的 `text` op 走 Unicode 注入，绕过输入法** ⇒ 永不触发候选窗。
   测候选窗只能用 `key` 逐个敲真键。
3. **判候选窗归属不要靠肉眼直觉，靠配色值比对** —— 微软拼音是蓝底横排，
   小狼毫的 endfield 是亮黄高亮 + 青序号，一眼可分。

## 下次要做的

- [ ] 校准任务栏开始按钮的物理像素坐标（当前 (30,1056) 点不中）
- [ ] 开始菜单里敲一次，与记事本那张逐项比对配色，得出「系统宿主是否按同一套 UI 渲染」
- [ ] 若需验背景图本身：把 `style/background_image` 指回 `endfield-light.png` 并**重新部署**
      （改 `weasel.custom.yaml` 后必须重新部署才进 build）
- [ ] 结论出来后再回 fxliang 那条
