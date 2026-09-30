# 猜词游戏协助（贴进新 chat 即可）

目标：用尽量少的次数命中。每轮只输出候选词 + 可执行脚本；我跑完把 `COPY_ME` 贴回来。不要寒暄、不要问是否开始、不要解释规则。

中文语义猜词：输入两字词，返回 0～100%，100% 即答案。**答案固定两个汉字**；不要猜单字、三字及以上、或带标点。
页面：`input[placeholder="任意猜一个词汇……"]`，按钮文案是「猜 测」（匹配时去掉空白再认「猜测」）。命中出现绿色条 `.ant-typography-success`，文案「X是正确的答案！」；帮助卡里的「猜盐」也是这种样式，不是命中。底部 `#标签` 只作参考：用一个这类的大词试一次，分数低就不要再猜这个方向。

## 你怎么做

看本局全部 `ROUND` 做决定，不要只看最后一轮。词必须现选，不要套固定表。每个词恰好两字，验证不同猜想，不要连出近义词。
第一轮输出完整 `guessWord` 并立刻调用；之后只改 `guessWord([...])`。页面刷新、`guessWord is not defined`、或贴回显示「已猜 0 次」时再注入一次。
已猜过的词（含 `null`、全部 ROUND 出现过的）本局不要再猜。`>=100` 或成功条出现：只确认命中词，停止出新词。一局只用一局的分，换题旧分作废。

## 分数怎么读

高分只说明常和答案一起出现，不说明答案属于这一类。用来比较的词要一样常见；反义词可以一边高一边低，不能拿来比较。
`null` 不是低分：这个词别再猜，但不要因此丢掉整条方向。

用 **本局最高分 H** 和 **与 H 的分差** 决策，不编故事。

| 分差（相对 H，两边都常用）           | 含义                                                                                                              |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| 低 15+                               | 丢掉这个词。要丢掉整类，需要两个常用词都低 15+。一个词低只否定这一个；描述反应或功能的词低，不单独丢掉那个方向   |
| 低不到 15                            | 这个方向还在                                                                                                      |
| 细词比大类词低 15+                   | 不要用更细的词去解释那个大类词，也不要再给下一个很接近的细词                                                     |
| 具体词高于类名或总称                 | 就停在这一层。补这一层还没猜的常用词；更宽的说法再低，就不要再往抽象猜                                           |
| 类名高、连续两个常用具体例子都低 15+ | 停止列举例子；高的是这一类，或某种特点，不是某一个                                                                |
| 目的、地点或人物已有两个常用词低 15+ | 这个方向丢掉。换个说法仍是同一个方向，不要再测                                                                    |

再看最近一轮把 H 抬高了没有。

| 最近一轮                             | 下一轮                                                       |
| ------------------------------------ | ------------------------------------------------------------ |
| 有词抬高 H                           | 顺着这个词。升幅 ≥10，或第一次跨过 70 / 90：先补这一层还没猜的词 |
| 没有抬高 H                           | 关掉这次猜的方向，回到上次把 H 抬高的那个词                      |
| 同一方向连续两轮都没抬高，或越猜越低 | 这个方向丢掉，换说法也不要再测                               |

## 每轮几个词

按最高分分段，边界算在左边：不到 50、50 到 69、70 到 89、90 及以上。70 分开始算接近答案。
**换方向优先**：连续两轮 H 提升 <3（H 不到 50 也算）；或大类词的内部已经测过且低 15+、H 仍 <70；或同一个字已经有 **3 个** ≥70 的词。同一类里多个高分却不是 100，只说明方向对了、词还没中，不要因此换方向。具体词高于类名或总称时，即使连续两轮不涨，也先补这一层还没猜的词。没抬高 H 的方向关掉，回到最近一次把 H 抬高的那个词。换方向时不要顺着大类再往下拆，也不要改猜无关的行业。

| 阶段   | 何时                  | n     | 本轮测什么                                                                                                                                                                                                  |
| ------ | --------------------- | ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 开局   | 还没有分              | **8** | 8 个互不相关的方向，必须跑完，`guessWord(..., false)`                                                                                                                                                       |
| 找方向 | H <50，还没到换方向   | **6** | 4 个新方向 + 2 个用来比较的词。H 是单独一个大类、内部还没测：改成 4 个其他大类 + 2 个这个大类里面的词（比较用大类，不用很具体的名字）。内部已测且低 15+：改走换方向                                     |
| 缩小   | H 50–69，还没到换方向 | **6** | 3 个新方向 + 3 个用来比较的词。前两名都是同一边的大类词：3 个新方向里必须有一个词测「谁在做 / 句子里的谁」                                                                                                |
| 接近   | H 70–89，这一类还没猜满 | **4** | 3 个旁边的词 + 1 个换个角度的词。一个 ≥80 且另一个不同类 ≥65，或两个不同类都 ≥70：必须有 1 个词是「这两边经常一起说的两字词」                                                                              |
| 换方向 | 见上                  | **4** | 3 个为什么 / 在哪 / 还有谁 + 1 个：补开局没测的方向，或两个高分方向合成的词，或再确认一次旧方向。具体词高于类名或总称时不走这一行                                                                          |
| 最后   | H ≥90                 | **3** | 2 个为什么 / 在哪 + 1 个收尾。不要猜工具、路线、地点上的小部件。多个具体近义词 ≥90 却不是 100：收尾补这一层还没猜的常用叫法；总称已经低于这些具体词，就不要再用总称。同一个字的感受词一个 ≥90、旁边还有 85+ 却不是 100：收尾改成相邻的性格或态度，不要再猜同一个字、来源、场景、身体反应 |

不要为了凑数加近义词。同一层还没猜的词不算凑数，一轮最多 2 个。

## 按最高分怎么猜

**开局（无分）**  
8 个词拉开，常用程度差不多，一个词一个方向：日常身份、大家都认识的具体名字、东西或媒介、地方、时间、地上景物、天上或天气、抽象想法或句子成分（后两项这轮选一个，另一个记下来后面补）。本轮被打断了，下一轮先补完这 8 个。开局全部 <30、后来出现 50 分以上的新方向：开局那些方向作废。

**H <50**  
新方向要多于用来比较的词。多个身份或头衔都在 40–50，更具体的称呼却低 15+：改猜有名有姓的人或作品名，不要再猜下一个身份。

**H 50–69**  
可以围着当前最高的主题，但每个词测的猜想要不同。单独一个行业大类领先、里面还没测：先比其他行业的大类词，再猜它里面的词。

**H ≥70**  
不要再开 6 个无关大类。先看具体还是笼统：具体词高于类名或总称，就留在这一层，一轮最多补 2 个还没猜的同层词，其余才是为什么、在哪、还有谁。类名更高，或两个常用例子低 15+：再离开「还是同一类东西」，改猜为什么、在哪、还有谁，或把两个高分方向合成一个词。不要用总称代替同层还没猜的词，也不要猜空话。
含同一个字、而且 ≥70 的词：有 1～2 个时，必须再试这个字的常用两字词（优先和另一个 ≥65 的不同类合成）；**有 3 个** 就不要再猜这个字。
H 停在 70–89 且连续两轮不涨：4 个词里至少 1 个来自开局还没测过的大类。具体词高于类名时，这个词改为同一层还没猜的词。
换方向时，同一场景里的场合、工具、品种、配料都已经低 15+：下一轮改成合成词或换个说法，不要再给下一个场景里的小部件。
做法或工具 ≥85：只留 1 个词确认旧做法，其余猜为什么、对谁、在哪。
书面的雅称或尊号很多：留一个日常口语词。
旧方向是身份：「还有谁」优先猜具体名字。

**H ≥90**  
不要猜工具、路线、地点上的小地方。具体叫法高于总称：最后一个词用还没猜的、同一层的常用词。总称不低于具体叫法时，才用总称。最近几轮没抬高 H，或总称越猜越低：最后一个词回到最高的具体词，补它同一层的叫法。同一个字的感受：改猜相邻的性格或态度。描述反应或功能的词 ≥80 仍不是 100：下一个词改相邻的性格或态度，不要解释「这个感受在做什么」。目的、地点、人物已经有两个常用词低 15+：不要换个说法再测。

## 输出格式（不要多写）

第一轮：

````
状态：前三名=无；阶段=开局；方向=先把大方向拉开；已排除=无；本轮要排除=答案落在哪一类

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
- `skipRestOnHighScore` 默认 true：某个词成了新的最高分，并且是第一次达到 70，或者比原来的最高分至少高出 10 分、自己也不低于 70 时，这一轮剩下的词不用再猜。只比原来高一点点时，后面的词继续猜。开局必须传 `false`。
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

现在开始第一轮。立刻给出状态句、8 个互不相关的词、完整注入模板，并以 `guessWord([...], false)` 调用。
