# 猜词游戏协助（贴进新 chat 即可）

目标：用尽量少的次数命中。每轮只输出候选词 + 可执行脚本；我跑完把 `COPY_ME` 贴回来。不要寒暄、不要问是否开始、不要解释规则。

中文语义猜词：输入两字词，返回 0～100%，100% 即答案。**答案固定两个汉字**；不要猜单字、三字及以上、或带标点。
页面：`input[placeholder="任意猜一个词汇……"]`，按钮文案是「猜 测」（匹配时去掉空白再认「猜测」）。命中出现绿色条 `.ant-typography-success`，文案「X是正确的答案！」；帮助卡里的「猜盐」也是这种样式，不是命中。底部 `#标签` 只当弱提示：用一个该类大词测一次，低就丢掉。

## 你怎么做

看本局全部 `ROUND` 做决定，不要只看最后一轮。词必须现选，不要套固定表。每个词恰好两字，验证不同猜想，不要连出近义词。
第一轮输出完整 `guessWord` 并立刻调用；之后只改 `guessWord([...])`。页面刷新、`guessWord is not defined`、或贴回显示「已猜 0 次」时再注入一次。
已猜过的词（含 `null`、全部 ROUND 出现过的）本局不要再猜。`>=100` 或成功条出现：只确认命中词，停止出新词。一局只用一局的分，换题旧分作废。

## 分数怎么读

高分只说明常和答案一起出现，不说明答案属于这一类。对照必须常用程度接近；反义词可以一边高一边低，不能当对照。
`null` 不是低分：这个词别再猜，但不要因此砍掉整条方向。

用 **本局最高分 H** 和 **与 H 的分差** 决策，不编故事。

| 分差（相对 H，两边都常用）           | 含义                                                                                                              |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| 低 15+                               | 丢掉这个成员。要丢掉整类，需要两个常用成员都低 15+。一个成员低只否定这一个；反应/功能描述低，不单独砍掉那条属性轴 |
| 低不到 15                            | 这条还活着                                                                                                        |
| 细词比宽词低 15+                     | 不要用更细的词去解释那个宽词，也不要再给第 N 个细近义                                                             |
| 类名高、连续两个常用具体品种都低 15+ | 停止枚举品种；高的是类或属性，不是某一个                                                                          |

## 每轮几个词

分数段左闭右开：`<50`、`50–69`、`70–89`、`≥90`。70 起算靠近。
**跳层优先**：连续两轮 H 提升 <3（H 不到 50 也算）；或同一类多个高分却不是 100；或宽词内部已测且低 15+ 而 H 仍 <70；或同一语素已有 **3 个** ≥70。跳层时不要拉二级轴、不要再开无关行业。

| 阶段   | 何时                | n     | 本轮测什么                                                                                                                                                                                                  |
| ------ | ------------------- | ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 定坐标 | 还没有分            | **8** | 8 个互斥方向，必须跑完，`guessWord(..., false)`                                                                                                                                                             |
| 找方向 | H <50，未跳层       | **6** | 4 新方向 + 2 对照。H 是单独宽主题且内部未测：改 4 个其他大主题 + 2 个该主题内部（对照用大主题，不用细名）。内部已测且低 15+：改走跳层                                                                       |
| 收窄   | H 50–69，未跳层     | **6** | 3 新方向 + 3 对照。前两名都是同一边的宽词：3 个新方向里必须有一枪「谁在行动 / 句子里的谁」                                                                                                                  |
| 收口   | H 70–89，该类未猜满 | **4** | 3 旁边 + 1 换角度。一条 ≥80 且另一条不同类 ≥65，或两条不同类都 ≥70：必须有 1 枪「两条线常并着说的两字词」                                                                                                   |
| 跳层   | 见上                | **4** | 3 个为什么/在哪/还有谁 + 1 个：补开局缺口，或两轴合成词，或确认旧方向                                                                                                                                       |
| 临门   | H ≥90               | **3** | 2 个为什么/在哪 + 1 收尾。不要猜工具、路线、地点零件。多个近义名字 ≥90 不是 100：收尾用两字合称，不要再换叫法。同根感受一个 ≥90、旁边 85+ 不是 100：收尾改相邻性格/态度，不要再猜同根、来源、场景、身体反应 |

不要为凑数加近义词。

## 按最高分怎么猜

**开局（无分）**  
8 枪拉开，常用程度差不多，一枪一类：日常身份、大家都认识的具体名字、东西或媒介、地方、时间、地上景物、天上或天气、抽象想法或句子成分（后两项这轮选一个，另一个记缺口后面补）。本轮打断了，下一轮先补完这 8 个。开局全员 <30、后来出现 50+ 新方向：开局轴作废。

**H <50**  
新方向多于对照。多个身份/头衔都在 40–50 而更细称呼低 15+：改猜有名有姓的人或作品名，不要第 N 个身份。

**H 50–69**  
可以围着领先主题，但每个词测不同猜想。单独一个行业宽词领先且内部未测：先比别的行业大词，再挖内部。

**H ≥70**  
不要再开 6 条无关大类。离开「还是同一类东西」，改猜为什么/在哪/还有谁，或两条高分线的合成词。不要第 N 个同类，不要空洞套话。
同一语素 ≥70 的词：1～2 个时必须再试该字的常用两字组合（优先和另一条 ≥65 的不同类线合成）；**3 个** 才封杀该语素。
H 停在 70–89 且连续两轮不涨：4 词里至少 1 个来自开局还没测过的大类。
跳层已在同一场景测过场合/工具/品种/配料且都低 15+：下一轮改合成或换说法，不要再给第 N 个场景零件。
做法/工具 ≥85：只留 1 枪确认旧做法，其余为什么/对谁/在哪。
书面雅称/尊号扎堆：留一枪日常口语。
旧方向是身份：「还有谁」优先具体名字。

**H ≥90**  
不要猜工具、路线、地点细节。同一对象多种叫法：合称。同根感受：相邻性格/态度。反应/功能词 ≥80 仍不是 100：下一枪相邻性格/态度，不要解释「这个感受在做什么」。

## 输出格式（不要多写）

第一轮：

````
状态：前三名=无；阶段=定坐标系；方向=先把大方向拉开；已排除=无；本轮要排除=答案落在哪一类

1. 词 — 验证…
…共 8 条

```javascript
window.guessWord = async (words, skipRestOnHighScore = true) => {
  ...完整模板...
};
guessWord(["词1","词2","词3","词4","词5","词6","词7","词8"], false);
````

第二轮起：

````
状态：前三名=词/分,词/分,词/分；阶段=…；方向=…；已排除=…；本轮要排除=…

1. 词 — 验证…
…条数必须等于本阶段词数

```javascript
guessWord(["词1", "词2"]);
````

```

`guessWord` 词表长度必须等于本轮词数。除第一轮或刷新外不要再贴函数体。

## 脚本要求

- 间隔 1 秒；「访问过快」则等 3 秒重试同一词。
- React value setter 写入输入框再点按钮：`textContent.replace(/\s/g, "").includes("猜测")`。用「已猜 N 次」确认提交。
- 跳过页面上已有分数的词。
- 命中：`.ant-typography-success` 且文案匹配 `是正确的答案`；或分数 `>=100`。成功条出现即记 100 并停止，写入 `COPY_ME`。不要用 `innerText` 搜「正确答案」，不要把帮助区「猜盐」当命中。
- `skipRestOnHighScore` 默认 true：某词刷新最高分，且（首次跨过 70，或比上一最高分高 ≥10 且自身 ≥70）时丢掉本轮剩余词。小步新高不要停。定坐标必须传 `false`。
- 开局已有成功条：直接 `COPY_ME` 并 return。
- `COPY_ME` 累积全部轮次。`ROUND` 带 `n=`；词按分数降序，`null` 最后。然后 `TOTAL`（优先页面「已猜 N 次」）、`TOP10`（已见最高分 10 词，不含 `null`）。用 `window.__guessRounds` 保存。刷新后需重新注入。示例：

```

COPY_ME
ROUND1 n=8
电影 62.51, 公园 53.76, 手机 48.56, 爱情 46.82, 老师 42.69, 春天 32.94, 跑步 26.21, 雨水 19.56
ROUND2 n=6
演员 70.12, 音乐 58.00, 游戏 55.00, 旅游 52.00, 美食 50.00, 电影院 48.00
TOTAL 14
TOP10
演员 70.12, 电影 62.51, 音乐 58.00, 游戏 55.00, 公园 53.76, 旅游 52.00, 美食 50.00, 手机 48.56, 电影院 48.00, 爱情 46.82

````

注入模板（仅第一轮或刷新后；之后只改词表）：

```javascript
window.guessWord = async (words, skipRestOnHighScore = true) => {
  const sleep = (ms) => new Promise((r) => setTimeout(r, ms));
  const pageText = () => document.body.innerText;

  const guessCount = () => Number(pageText().match(/已猜\s*(\d+)\s*次/)?.[1] ?? -1);

  const winner = () => {
    const bar = [...document.querySelectorAll(".ant-typography-success")].find((el) => /是正确的答案/.test(el.textContent));
    if (!bar) return null;
    return bar.textContent.match(/(\S+?)是正确的答案/)?.[1] ?? "";
  };

  const boardScores = () => {
    const scores = new Map();
    for (const [, word, value] of pageText().matchAll(/(\S{1,10})\s+(\d+(?:\.\d+)?)%/g)) {
      const score = Number(value);
      if (score > (scores.get(word) ?? -1)) scores.set(word, score);
    }
    return scores;
  };
  const scoreOf = (word) => boardScores().get(word) ?? null;

  const typeAndSubmit = async (word) => {
    const input =
      document.querySelector('input[placeholder="任意猜一个词汇……"]') || document.querySelector("input.ant-input");
    if (!input) return false;
    input.focus();
    Object.getOwnPropertyDescriptor(HTMLInputElement.prototype, "value").set.call(input, word);
    input.dispatchEvent(new Event("input", { bubbles: true }));
    input.dispatchEvent(new Event("change", { bubbles: true }));
    await sleep(60);
    [...document.querySelectorAll("button")].find((b) => b.textContent.replace(/\s/g, "").includes("猜测"))?.click();
    await sleep(1000);
    return true;
  };

  const guess = async (word) => {
    const known = scoreOf(word);
    if (known !== null) return known >= 100 ? 100 : known;
    for (let attempt = 0; attempt < 4; attempt++) {
      const before = guessCount();
      if (!(await typeAndSubmit(word))) break;
      if (pageText().includes("访问过快")) {
        await sleep(3000);
        continue;
      }
      if (winner() !== null || guessCount() !== before) break;
      await sleep(1000);
    }
    const score = scoreOf(word);
    if (winner() !== null || (score ?? 0) >= 100) return 100;
    return score;
  };

  const report = (rows) => {
    const hit = winner();
    const answer = hit === null ? null : hit || rows.at(-1)?.word;
    const round = (answer ? [...rows.filter((r) => r.word !== answer), { word: answer, score: 100 }] : rows).sort(
      (a, b) => (b.score ?? -1) - (a.score ?? -1)
    );
    const history = (window.__guessRounds ||= []);
    history.push(round);

    const line = (rows) => rows.map((r) => `${r.word} ${r.score ?? "null"}`).join(", ");
    const bestOf = new Map();
    for (const r of history.flat()) if (r.score !== null && r.score > (bestOf.get(r.word) ?? -1)) bestOf.set(r.word, r.score);
    const top10 = [...bestOf].sort((a, b) => b[1] - a[1]).slice(0, 10).map(([word, score]) => ({ word, score }));
    const total = guessCount() >= 0 ? guessCount() : history.flat().length;

    console.log(
      [
        "COPY_ME",
        ...history.map((rows, i) => `ROUND${i + 1} n=${rows.length}\n${line(rows)}`),
        `TOTAL ${total}`,
        "TOP10",
        line(top10),
      ].join("\n")
    );
  };

  if (winner() !== null) return report([{ word: winner() || "命中", score: 100 }]);

  const rows = [];
  let best = Math.max(0, ...boardScores().values());
  for (const word of words) {
    const score = await guess(word);
    rows.push({ word, score });
    console.log(`${word} ${score ?? "未上榜"}% (已猜 ${guessCount()})`);
    if (score === 100) break;
    const skipRest = skipRestOnHighScore && score >= 70 && score > best && (best < 70 || score - best >= 10);
    best = Math.max(best, score ?? 0);
    if (skipRest) {
      console.log("跳过剩余", word, score);
      break;
    }
  }
  report(rows);
};
````

## 开局

现在开始第一轮。立刻给出状态句、8 个互斥探针、完整注入模板，并以 `guessWord([...], false)` 调用。
