# 猜词游戏协助（贴进新 chat 即可）

目标：用尽量少的猜测次数命中答案。你每轮只输出候选词 + 可执行脚本；我跑脚本后把 `COPY_ME` 贴回来。不要寒暄、不要问我是否开始、不要解释游戏规则。

我在玩一个中文语义猜词游戏（输入词语，返回 0～100% 的相关度，100% 即答案）。
页面有输入框 `input[placeholder="任意猜一个词汇……"]` 和「猜测」按钮，猜测结果以「词 + 百分比」列在页面上。
命中时会出现绿色成功条（`.ant-typography-success`，文案为「X是正确的答案！」），命中词不一定会出现在榜单上，也不一定有 `100.00%`。

## 你的职责

1. 按阶段给出本轮候选词（数量见下表），每个词附一句短理由（在验证什么假设）。
2. 探轴阶段词必须互斥；对照/跳层阶段允许同一话语场，但每个词必须对应不同假设（并列 / 上位 / 内部 / 新轴），禁止近义复读。
3. 第一轮输出完整注入脚本（定义 `guessWord` 并立刻调用）。之后每轮只输出一行 `guessWord([...])`，不要重复函数体。页面刷新或报 `guessWord is not defined` 时再给一次完整注入。
4. 决策必须基于 `BOARD` 全榜（本局全部已见分数），不要只看本轮 `ROUND`。持续记住：黑名单、已否轴、Top3、当前一条待证伪假设。
5. 我把 `COPY_ME` 贴回来后出下一轮。已猜过的词不要重复。

## 每轮猜几个

以本局 `BOARD` 最高分为准。70 落在「收口」；边界按下表左闭右开理解：`<50`、`50–69`、`70–89`、`≥90`。  
「同簇全高无满分」或「连续两轮最高分提升都 &lt; 3」时，走 **跳层**，不要走收口。

| 阶段 | 条件 | 本轮词数 | 配比 |
|------|------|----------|------|
| 定坐标系 | 开局，尚无分数 | **8** | 固定词，见下；本轮禁止早停 |
| 拉二级轴 | 最高分 &lt; 50 | **6** | 4 新轴 + 2 对照 |
| 收窄 | 最高分 50–69 | **6** | 3 新轴 + 3 对照 |
| 收口 | 最高分 70–89，且尚未判定同簇饱和 | **4** | 3 邻域对照 + 1 跳层探针 |
| 跳层 | 同簇实例全高无满分，或连续两轮最高分提升 &lt; 3 | **4** | 3 并列/上位 + 1 旧簇封口 |
| 临门 | 最高分 ≥ 90 | **3** | 几乎全是对照，最多 1 个跳层探针 |

不要为凑数加近义词。宁少勿滥。

最高分是 **领域/行业标签**（如科技、历史、经济）时：拉二级轴/收窄改为 **4 领域对照 + 2 该标签内部**。领域对照从尚未猜过的槽位里取：`交通 教育 农业 文化 体育 医疗 能源`。不要先钻该标签的内部细分（航天、芯片、AI）。

## 猜词策略（必须遵守）

- **高分是共现邻居，不是同类标签。** 某类实例分高，不代表答案属于该类。泰山/黄山/华山都 85+，答案可以是长城。科技 86 也可以只是「领域词列表」里的一张牌，答案是旅游。
- **先定坐标系，再填格子。** 开局不要枚举同类实例。通名可能只有 30～40，仍要先拉开轴，再用更具体但仍互斥的轴比较。定坐标系若被打断，下一轮先补完未猜的那 8 个探针，再开新轴。
- **领域标签高，先换行业槽，再挖内部。** 科技/历史/经济这类通名领先时，对照必须是其他行业（交通、教育、农业、文化），不是创新/政策/中国/发展，也不是航天/人工智能。连续两轮最高分不动、且新词都落在 75–85：认定行业共现团，下一轮必须换未测行业槽，禁止再给该团细分。
- **同级实例全高却无满分，就跳层。** 跳到同一话语场里的其他角色（标题/句子里常一起出现、但不是同一 IS-A），不要在簇里继续枚举第 N 个成员。名山簇全体 80+ 且无满分：改测山河/城墙/关隘等并列，而不是第 N 座山。若饱和簇本身就是行业通名（科技/教育/农业都 80+），并列枪必须是未测行业槽（交通/文化/体育/文旅），旧簇封口也用未测领域；禁止改测创新/现代化/国防/政策/中国/发展。
- **顺着最高分专名往里钻，若局部掉分就停。** 景点、别称、所在省若明显更低，说明高分是共现不是从属，不要继续挖该专名内部。
- **上位通名可以低于实例。** 通名低、实例高，更要跳出该簇，不代表答案是「这一类里的某个」。
- **同频才能比方向。** 近义词会因词频差 20 分，对照词要选常用程度接近的。
- **类别词 vs 专名要分开测。** 具体名字普遍高于类别词，答案多半是专名；反之是通名。专名高只说明答案常和这些专名一起出现。
- **临门先测目的/场景，后测载具。** 出行/交通/运输通名已高时，下一枪用旅游/旅行这类目的词，不要先枚举高铁/公路/汽车。载具或线路掉分，立刻停挖方式。
- **标签只是弱提示**，用一个类名词验证，低就丢掉。
- **`null` 不是低分。** 未上榜只说明没进可见列表：加入黑名单不再猜，但不得据此砍轴。
- **一局只用一局的数据**，换题后旧分作废。
- 每轮开头先用一句话说明：Top3、阶段、方向、本轮要证伪的假设、已否轴。

## 决策规则

- 第一轮固定这 8 个，不要换词：`人物 器物 历史 科技 影视 食物 山 抽象`。调用 `guessWord(words, false)`，禁止早停。
- 最高分 &lt; 50：继续探索，新轴多于对照。
- 最高分 ≥ 70 且未跳层：对照多于新轴；禁止再开 6 条无关大类。
- 只根据分数差决策，不编故事。相对最高分掉 15+ **且对照词常用程度接近** 才丢掉该方向；若对照词明显更生僻，先换一个同频词再测，不要直接砍轴。
- 跳层时 3 枪必须离开旧簇（并列或上位），只留 1 枪封口旧簇；禁止再给第 N 个同类实例。行业通名饱和时，离开 = 换行业槽，不是上政策层。
- 专名内部掉分：下一轮改测并列概念，不要继续挖景点/别称/行政区。
- 已猜过的词（含 `null`）本局黑名单。`BOARD` 上的词也算已猜。
- 若某一词 `>= 100` 或出现成功条，停止出新词，只确认命中词。

## 你每轮的输出格式（严格按此，不要多写）

第一轮：

```
状态：Top3=无；阶段=定坐标系；方向=拉开轴；已否轴=无；本轮证伪=答案落在哪条大类/二级轴

1. 词 — 验证…
…共 8 条

```javascript
window.guessWord = async (words, earlyStop = true) => {
  ...完整模板...
};
guessWord(["人物","器物","历史","科技","影视","食物","山","抽象"], false);
```
```

第二轮起（函数已在页面上）：

```
状态：Top3=词/分,词/分,词/分；阶段=…；方向=…；已否轴=…；本轮证伪=…

1. 词 — 验证…
…条数必须等于本阶段词数

```javascript
guessWord(["词1","词2"]);
```
```

`guessWord` 的词表长度必须等于本轮词数。除第一轮完整注入外，不要再贴函数体。

## 脚本要求

- 间隔 1 秒猜一次；命中「访问过快」则等 3 秒重试同一个词。
- 通过 React 的 value setter 写入输入框，再点按钮；用「已猜 N 次」校验是否真的提交，未提交就重试。
- 跳过页面上已有分数的词。
- **命中判定（必须同时遵守）：**
  - 用 `.ant-typography-success` 且文案匹配 `是正确的答案` 判定命中；不要用 `document.body.innerText` 搜「正确答案」（帮助文字会误判）。
  - 分数 `>= 100` 也算命中。
  - 成功条出现即记 100 并停止；不要等榜单出现该词。
- **早停：** `earlyStop === true`（默认）时，本词成为新高分，且（跨过 70，或比上一最高分高出 ≥10 且自身 ≥70）时停止本轮剩余词。85→86 这种小步新高不要停。定坐标系必须 `earlyStop === false`。
- 开局若已有成功条，直接 `COPY_ME` 并 return。
- 结尾输出 `COPY_ME`：先 `ROUND`（本轮词和分数），再 `BOARD`（页面可见全部词和分数），每行 `词<TAB>分数`。

注入模板（仅第一轮或刷新后输出；之后只改 `guessWord([...])` 的词表）：

```javascript
window.guessWord = async (words, earlyStop = true) => {
  const DELAY_MS = 1000;
  const sleep = (ms) => new Promise((r) => setTimeout(r, ms));
  const getInput = () =>
    document.querySelector('input[placeholder="任意猜一个词汇……"]') ||
    document.querySelector("input.ant-input");
  const getButton = () =>
    [...document.querySelectorAll("button")].find((b) => b.textContent.replace(/\s/g, "").includes("猜测"));
  const getCount = () => {
    const m = document.body.innerText.match(/已猜\s*(\d+)\s*次/);
    return m ? Number(m[1]) : -1;
  };
  const tooFast = () => document.body.innerText.includes("访问过快");
  const hitInfo = () => {
    const el = [...document.querySelectorAll(".ant-typography-success")].find((n) =>
      /是正确的答案/.test(n.textContent || "")
    );
    if (!el) return null;
    const m = (el.textContent || "").match(/(\S+)是正确的答案/);
    return { word: m ? m[1] : null, text: el.textContent.trim() };
  };
  const boardMap = () => {
    const map = new Map();
    for (const r of document.body.innerText.matchAll(/(\S{1,10})\s+(\d+\.\d{2})%/g)) {
      const s = Number(r[2]);
      if (!map.has(r[1]) || s > map.get(r[1])) map.set(r[1], s);
    }
    return map;
  };
  const scoreOf = (w) => {
    const s = boardMap().get(w);
    return s === undefined ? null : s;
  };
  const bestScore = () => {
    const vals = [...boardMap().values()];
    return vals.length ? Math.max(...vals) : 0;
  };
  const setValue = (el, v) => {
    Object.getOwnPropertyDescriptor(HTMLInputElement.prototype, "value").set.call(el, v);
    el.dispatchEvent(new Event("input", { bubbles: true }));
    el.dispatchEvent(new Event("change", { bubbles: true }));
  };
  const dump = (round) => {
    const board = [...boardMap().entries()].sort((a, b) => b[1] - a[1]);
    const roundText = round.map((r) => `${r.word}\t${r.score ?? "null"}`).join("\n");
    const boardText = board.map(([w, s]) => `${w}\t${s}`).join("\n");
    console.log("COPY_ME\nROUND\n" + roundText + "\nBOARD\n" + boardText);
  };

  const results = [];
  const already = hitInfo();
  if (already) {
    console.log("已命中", already.word);
    dump([{ word: already.word, score: 100 }]);
    return;
  }
  let bestBefore = bestScore();
  for (const word of words) {
    const had = scoreOf(word);
    if (had !== null) {
      results.push({ word, score: had });
      console.log(`${word} ${had}% (已有)`);
      if (had >= 100) {
        console.log("命中", word);
        break;
      }
      continue;
    }
    let ok = false;
    for (let retry = 0; retry < 4 && !ok; retry++) {
      const input = getInput();
      if (!input) break;
      const before = getCount();
      input.focus();
      setValue(input, word);
      await sleep(60);
      getButton()?.click();
      await sleep(DELAY_MS);
      if (tooFast()) {
        await sleep(3000);
        continue;
      }
      if (hitInfo()) {
        ok = true;
        break;
      }
      if (getCount() === before) {
        await sleep(DELAY_MS);
        continue;
      }
      ok = true;
    }
    const hit = hitInfo();
    const score = hit && (hit.word === word || hit.word === null) ? 100 : scoreOf(word);
    results.push({ word, score: score ?? (hit ? 100 : null) });
    console.log(`${word} ${score ?? (hit ? 100 : "未上榜")}% (已猜 ${getCount()})`);
    if ((score ?? 0) >= 100 || hit) {
      console.log("命中", hit?.word || word);
      break;
    }
    if (earlyStop && score !== null && score > bestBefore) {
      const crossed = bestBefore < 70 && score >= 70;
      const jump = score - bestBefore >= 10 && score >= 70;
      bestBefore = score;
      if (crossed || jump) {
        console.log("早停", word, score);
        break;
      }
    } else if (score !== null && score > bestBefore) {
      bestBefore = score;
    }
  }
  dump(results);
};
```

## 开局

现在开始第一轮。立刻给出状态句、固定 8 个探针、完整注入模板，并以 `guessWord([...], false)` 调用。
