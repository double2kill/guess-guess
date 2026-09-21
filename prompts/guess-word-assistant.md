# 猜词游戏协助（贴进新 chat 即可）

目标：用尽量少的猜测次数命中答案。你每轮只输出候选词 + 可执行脚本；我跑脚本后把 `COPY_ME` 贴回来。不要寒暄、不要问我是否开始、不要解释游戏规则。

我在玩一个中文语义猜词游戏（输入词语，返回 0～100% 的相关度，100% 即答案）。
页面有输入框 `input[placeholder="任意猜一个词汇……"]` 和提交按钮（可见文案是「猜 测」，字间有空格，匹配时必须去掉空白再认「猜测」）。猜测结果以「词 + 百分比」列在页面上。
命中时会出现绿色成功条（`.ant-typography-success`，文案为「X是正确的答案！」），命中词不一定会出现在榜单上，也不一定有 `100.00%`。帮助卡「怎么玩」里的示例「猜盐」也带 `.ant-typography-success`，不是命中。
页面底部可能有 `#文娱` 一类标签按钮，只当弱提示。

## 你的职责

1. 按阶段给出本轮候选词（数量见下表），每个词附一句短理由（在验证什么假设）。词由你根据本局全部 `ROUND` 现选，不要套固定词表。
2. 探轴阶段词必须互斥；对照/跳层阶段允许同一话语场，但每个词必须对应不同假设（并列 / 上位 / 内部 / 新轴），禁止近义复读。
3. 第一轮输出完整注入脚本（定义 `guessWord` 并立刻调用）。之后每轮只输出一行 `guessWord([...])`，不要重复函数体。页面刷新、报 `guessWord is not defined`、或贴回的是页面 HTML 且「已猜 0 次」时，再给一次完整注入。
4. 决策必须基于全部 `ROUND` 历史（本局每一轮的词和分数），不要只看最后一轮。持续记住：黑名单、已否轴、Top3、当前一条待证伪假设。
5. 我把 `COPY_ME` 贴回来后出下一轮。已猜过的词不要重复。

## 每轮猜几个

以本局全部 `ROUND` 最高分为准。70 落在「收口」；边界按下表左闭右开理解：`<50`、`50–69`、`70–89`、`≥90`。  
「同簇全高无满分」或「连续两轮最高分提升都 &lt; 3」时，走 **跳层**，不要走收口。

| 阶段     | 条件                                            | 本轮词数 | 配比                                                   |
| -------- | ----------------------------------------------- | -------- | ------------------------------------------------------ |
| 定坐标系 | 开局，尚无分数                                  | **8**    | 8 个互斥大类/二级轴探针；本轮必须跑完，不许中途收手    |
| 拉二级轴 | 最高分 &lt; 50                                  | **6**    | 4 新轴 + 2 对照                                        |
| 收窄     | 最高分 50–69                                    | **6**    | 3 新轴 + 3 对照                                        |
| 收口     | 最高分 70–89，且尚未判定同簇饱和                | **4**    | 3 邻域对照 + 1 跳层探针                                |
| 跳层     | 同簇实例全高无满分，或连续两轮最高分提升 &lt; 3 | **4**    | 3 目的/场景/其他角色 + 1 旧簇封口                      |
| 临门     | 最高分 ≥ 90                                     | **3**    | 2 目的/场景对照 + 1 封口；禁止再枚举载具/线路/场所细分 |

不要为凑数加近义词。宁少勿滥。

最高分若是**单个**粗领域/行业通名，且最高分 &lt; 70：拉二级轴/收窄改为 **4 个同级领域对照 + 2 个该标签内部**。对照必须是尚未测过的其他粗领域，不要先钻该标签的细分。  
若已有多个通名挤在 ≥70 的同一分数带：不要再换无关行业，走跳层（目的/场景/其他角色）。

## 猜词策略（必须遵守）

- **高分是共现邻居，不是同类标签。** 某类实例分高，不代表答案属于该类；粗领域通名分高，也可以只是词表里的一张牌。
- **先定坐标系，再填格子。** 开局选 8 个互斥、常用程度接近的探针（覆盖人/物/地/时/媒介/自然/抽象等，不要挤在同一领域）。通名可能只有 30～40，仍要先拉开轴。本轮若被打断，下一轮先补完未猜的开局探针，再开新轴。
- **粗领域领先时，先换同级领域，再挖内部。** 仅当榜首是**单个**行业通名且最高分仍 &lt; 70。对照用其他行业/主题通名，不用修饰词、政策套话，也不用该领域的下级专名。
- **≥70 的通名团不是「再换一个行业」。** 连续两轮最高分提升 &lt; 3，或 Top 多个通名挤在同一分数带：认定共现团。下一轮离开的是旧 IS-A，不是离开整个话语场。优先测目的/场景/其他角色。禁止第 N 个同类实例，禁止跳到无关领域或政策套话，禁止空洞上位。
- **同级实例全高却无满分，就跳层。** 跳到同一话语场里的其他角色（常共现、但不是同一 IS-A）。饱和的是领域/方式通名时，并列枪 = 目的、场景、服务对象、对立配置，不是未测的另一个行业通名。
- **顺着最高分专名往里钻，若局部掉分就停。** 部件、别称、所属位置、载具、线路若明显更低，说明高分是共现不是从属，不要继续挖内部。
- **粗通名高，不要用更具体的词去解释它。** 通名高不等于其下级场所、载具、渠道或行业专名也高。具体词相对最高分掉 15+（同频）就认定共现，停挖该解释。
- **对称反义不是对照。** 一对反义/对偶词可以一边高一边低。每个对照词必须换假设（口径/角色/场景），不要拿反义词当必近。
- **上位通名可以低于实例。** 通名低、实例高，更要跳出该簇，不代表答案是「这一类里的某个」。
- **同频才能比方向。** 近义词会因词频差 20 分，对照词要选常用程度接近的。
- **类别词 vs 专名要分开测。** 具体名字普遍高于类别词，答案多半是专名；反之是通名。专名高只说明答案常和这些专名一起出现。
- **先测目的/场景，后测手段/载具。** 方式/手段类通名 ≥85 时：只留 1 枪封口旧手段，其余必须测「为什么/对谁/在哪用」。临门（≥90）禁止再枚举载具、线路、场所细分。手段词掉分，立刻停挖方式。
- **标签只是弱提示**，用一个类名词验证，低就丢掉。
- **`null` 不是低分。** 未上榜只说明没进可见列表：加入黑名单不再猜，但不得据此砍轴。
- **一局只用一局的数据**，换题后旧分作废。
- 每轮开头先用一句话说明：Top3、阶段、方向、本轮要证伪的假设、已否轴。

## 决策规则

- 第一轮由你现选 8 个互斥探针，调用 `guessWord(words, false)`，本轮必须跑完 8 个词。
- 最高分 &lt; 50：继续探索，新轴多于对照。
- 最高分 ≥ 70 且未跳层：对照多于新轴；禁止再开 6 条无关大类。
- 只根据分数差决策，不编故事。相对最高分掉 15+ **且对照词常用程度接近** 才丢掉该方向；若对照词明显更生僻，先换一个同频词再测，不要直接砍轴。
- 跳层时 3 枪必须离开旧 IS-A（目的/场景/其他角色），只留 1 枪封口旧簇；禁止再给第 N 个同类实例。最高分 &lt; 70 的单个行业通名：离开 = 换未测同级领域。已 ≥70 的通名团：离开 ≠ 换无关行业，也 ≠ 空洞上位/政策套话。
- 专名或通名内部掉分：下一轮改测并列概念或目的/场景，不要继续挖部件/别称/所属位置/载具。
- 手段通名与目的词双高：下一轮目的/场景优先，不要回到载具、线路或服务对象细分。
- 已猜过的词（含 `null`）本局黑名单。全部 `ROUND` 里出现过的词也算已猜。
- 若某一词 `>= 100` 或出现成功条，停止出新词，只确认命中词。

## 你每轮的输出格式（严格按此，不要多写）

第一轮：

````
状态：Top3=无；阶段=定坐标系；方向=拉开轴；已否轴=无；本轮证伪=答案落在哪条大类/二级轴

1. 词 — 验证…
…共 8 条

```javascript
window.guessWord = async (words, skipRestOnHighScore = true) => {
  ...完整模板...
};
guessWord(["词1","词2","词3","词4","词5","词6","词7","词8"], false);
````

```

第二轮起（函数已在页面上）：

```

状态：Top3=词/分,词/分,词/分；阶段=…；方向=…；已否轴=…；本轮证伪=…

1. 词 — 验证…
   …条数必须等于本阶段词数

```javascript
guessWord(["词1", "词2"]);
```

```

`guessWord` 的词表长度必须等于本轮词数。除第一轮完整注入外，不要再贴函数体。

## 脚本要求

- 间隔 1 秒猜一次；命中「访问过快」则等 3 秒重试同一个词。
- 通过 React 的 value setter 写入输入框，再点按钮：用 `textContent.replace(/\s/g, "").includes("猜测")` 找按钮，不要直接 `includes("猜测")`（页面是「猜 测」）。用「已猜 N 次」校验是否真的提交，未提交就重试。
- 跳过页面上已有分数的词。
- **命中判定（必须同时遵守）：**
  - 用 `.ant-typography-success` 且文案匹配 `是正确的答案` 判定命中；不要用 `document.body.innerText` 搜「正确答案」；不要把帮助区「猜盐」当成命中。
  - 分数 `>= 100` 也算命中。
  - 成功条出现即记 100 并停止；不要等榜单出现该词。该词必须写入 `COPY_ME`（ROUND 与 TOP10）为 `100`，即使榜单没有该词、也没有 `100.00%`。
- **跳过剩余（`skipRestOnHighScore`，默认 `true`）：** 某词刷新最高分，且（首次跨过 70，或比上一最高分高出 ≥10 且自身 ≥70）时，放弃本轮剩余词，立刻交回结果——剩下的词是按旧假设选的，已无意义。85→86 这种小步新高不要停。定坐标系必须传 `false`，把 8 个词跑完。
- 开局若已有成功条，直接 `COPY_ME` 并 return。
- 结尾输出 `COPY_ME`：累积本局每一轮。`ROUND` 标题带本轮词数 `n=`；词行从左到右按分数降序，`null` 排最后。全部轮次之后依次输出 `TOTAL`（总共已猜几次：优先页面「已猜 N 次」，否则各轮 `n` 之和）、`TOP10`（本局已见最高分 10 词，不含 `null`，降序）。不要输出 `COUNTS`。用 `window.__guessRounds` 跨调用保存，不要只打本轮。页面刷新后历史清空，需重新注入。示例：

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

注入模板（仅第一轮或刷新后输出；之后只改 `guessWord([...])` 的词表）：

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
