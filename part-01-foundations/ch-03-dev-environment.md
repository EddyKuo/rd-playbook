---
chapter: 3
part: I
title: 開發環境與本機工作流
slug: dev-environment
agent: RD
domain_case: CASE-ECM-003
status: draft
word_count_target: 5500
---

# 第 3 章｜開發環境與本機工作流
## ⸺ 在「它在我機器上能跑」成為笑點之前,先把環境問題擋在自己這一關

> **前置閱讀**:[第 2 章｜讀懂一份陌生程式碼](./ch-02-reading-unfamiliar-code.md)
> **下游章節**:[第 4 章｜版本控制策略](./ch-04-version-control.md)

## 3.1 共感現場:那個週五下午的環境噩夢

你可能也遇過這種情況——但那次我是親眼看到的。

我帶過一個年輕工程師,就叫她小雅吧。那時她在一家做電商後台的公司 ShopNova 負責維護訂單管理服務。週五下午三點,她拉了一份 PR 下來,想在本機確認一個同事修的促銷計算邏輯。但 `npm install` 裝了十五分鐘,中途報了幾個奇怪的 peer dependency 警告;接著資料庫連不上,因為她本機的 PostgreSQL 是 14,而 CI 跑的是 17;好不容易把服務起來了,測試又跑不過,錯誤訊息指向一個環境變數找不到——那個變數在她的 `.env` 裡沒有,只有在團隊內部的 Confluence 某頁藏著,沒有人告訴她。

她跑來問我的時候,快五點了,臉上帶著那種努力很久但說不清楚自己卡在哪的疲憊。

這件事算不上一個大事故,但它其實每天都在世界各地的工程師身上發生。不同的版本、不同的設定、不同的 OS,每一個細節都可能悄悄讓「本機能跑」和「CI 能跑」成為兩個不同的宇宙。花在對齊環境上的時間不會出現在 sprint board 上,不會被計入技術債,但它每天都在偷走工程師最寶貴的那樣東西——**開始工作的動力**。

環境越難上手,人就越不願意多跑一次測試、多拉一次最新的 main、多試一個新的工具。這不是態度問題,是人對摩擦的自然反應。順著這個道理,讓我們把環境問題看得更清楚一點。

## 3.2 真正的問題:「它在我機器上能跑」不只是笑話

我們把小雅那個下午慢慢拆開來看。

表面上,那是「版本不一致」「文件沒更新」「環境變數沒交接」三個問題。但這三件事的根源其實是同一個:**開發環境的狀態只存在於每個人的腦袋或機器裡,沒有被版本化。**

當環境的定義只是口耳相傳,或者藏在一份兩年前的 Onboarding 文件裡,那麼「你的環境」和「CI 的環境」之間就會因為時間、OS、安裝順序的細微差異,慢慢走向不同的平行宇宙。問題不是哪個人出錯了,而是沒有一個機制讓環境的狀態可以被**重現**——就像程式碼可以從 git 重現一樣。

也就是說,可重現環境(Reproducible Environment)不是一個好習慣,而是工程的基礎設施。它解決的不只是「新人很難 onboard」,它也解決「今天能跑、明天不能跑」「你機器能跑、他機器不能跑」「本機能跑、CI 不能跑」這一整族問題。

順著這個道理,問題就變得更清晰了:我們需要的,不只是一份好的 README,而是讓環境的規格本身成為版本控制的一部分。

這就帶出了第二個面向:就算環境一致了,**回饋迴圈的速度**依然決定了你每天真正能做多少有效的工作。小雅等了那麼久才能開始確認那個 PR,不只浪費了時間,也打斷了她的思維流。

快速回饋(Fast Feedback)的核心是:你做了一個改動,能多快看到結果?如果這個等待超過三十秒,人就會切走做別的事情;切走了就代表同時在飛兩件事,上下文切換的成本非常昂貴。從這個角度看,環境設定得好不好,幾乎直接決定了你的程式碼會被多仔細地測試——因為測試快、人才願意測。

## 3.3 一起做判斷:三層環境決策

那麼,面對「環境」這件事,有哪些角度可以幫你理清要做什麼?

我想用三個層次來說明,每一層解決的問題不同,你可以根據團隊的現況決定從哪一層開始。

```mermaid
flowchart TD
  L1["Layer 1\n讓環境可重現\n(Reproducible)"]:::cold
  L2["Layer 2\n讓回饋迴圈快\n(Fast Feedback)"]:::cold
  L3["Layer 3\n讓環境問題在上線前被擋住\n(Shift Left)"]:::cold

  L1 --> L2 --> L3

  L1 --- T1["做法:容器化(Docker Compose)\n或 devcontainer\n或 mise/nix"]:::goal
  L2 --- T2["做法:hot reload / watch mode\n/ 最小啟動子集 / 本機 stub"]:::goal
  L3 --- T3["做法:本機跑 CI 子集\n/ pre-commit hook\n/ env-check 腳本"]:::goal

  classDef hot  fill:#fee,stroke:#c33
  classDef cold fill:#eef,stroke:#36c
  classDef goal fill:#efe,stroke:#3a3
```

三層之間的關係是遞進的:Layer 1 讓你的環境「可以跑」;Layer 2 讓你的環境「快到願意用」;Layer 3 讓你的環境「能攔截問題」。三層都不是一次性的設定,而是日常工作流程的一部分。先把 Layer 1 穩定下來,再往上疊,不要跳層。

### 3.3.1 Layer 1:讓環境可重現

可重現環境的目標是:**不論是誰的機器、什麼時間點,執行同一份設定,都能跑起相同的服務。**

做法有幾種,依複雜度排列:

| 方法 | 適合情境 | 主要優點 | 主要代價 |
|---|---|---|---|
| `.tool-versions` / `mise`(版本管理工具) | 語言版本管理為主 | 輕量、速度快 | 無法涵蓋 DB/外部服務 |
| `docker-compose.yml` | 需要 DB、快取、第三方服務 | 接近生產、隔離性好 | 初次啟動較慢 |
| `devcontainer.json`(VS Code / GitHub Codespaces) | 團隊混用 OS、CI 一致性要求高 | 連 IDE 設定都版控化 | 需要 Docker;設定略複雜 |
| Nix / Flake | 極高可重現性要求 | 幾乎完美的可重現 | 學習曲線陡、非主流 |

大多數 web 服務的後端專案,從 `docker-compose.yml` 開始是最划算的起點。它能同時解決「本機 PostgreSQL 版本不對」和「Redis 沒裝」這兩類問題,而且 CI 環境也可以用同一份 compose 設定來跑 integration test。

以 ShopNova 的 `order-service` 為例,一份夠用的 `docker-compose.yml` 大概長這樣:

```yaml
# docker-compose.yml (docker compose 2.x)
services:
  db:
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: orders_dev
      POSTGRES_PASSWORD: devpassword
      POSTGRES_DB: orders_dev
    ports:
      - "5432:5432"
    volumes:
      - orders_pgdata:/var/lib/postgresql/data
    profiles: [dev, full]

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    profiles: [dev, full]

  kafka:
    image: confluentinc/cp-kafka:7.6.0
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
    ports:
      - "9092:9092"
    depends_on: [zookeeper]
    profiles: [full]         # 只在跑 integration test 時才拉起

  zookeeper:
    image: confluentinc/cp-zookeeper:7.6.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
    profiles: [full]

volumes:
  orders_pgdata:
```

幾個值得注意的細節:

**`profiles` 欄位**是這份 compose 最重要的設計決策。日常開發只需要 `--profile dev`,把 PostgreSQL 17 和 Redis 7 跑起來;跑 integration test 才用 `--profile full`,連 Kafka 也一起拉。這樣「啟動時間」從五分鐘縮到三十秒,讓人願意每天用,而不是嫌慢就跳過。

**版本要精確到 minor tag**:用 `postgres:17-alpine` 而不是 `postgres:latest`。`latest` 今天是 17,明天可能是 18,而你根本不知道什麼時候換了。寫死版本,讓「更新版本」這件事成為一個有意識的 PR,而不是一個安靜的意外。

**同時搭配 `.tool-versions`** 管理 Node.js 版本:

```toml
# .tool-versions (mise / asdf 格式)
nodejs 22.4.0
pnpm 9.12.0
```

`mise install` 一行命令,讓所有人的 Node 版本對齊。這兩份檔案加在一起——`docker-compose.yml` 管外部依賴、`.tool-versions` 管執行環境——就能涵蓋小雅那天遇到的絕大部分問題。

順著這個道理,有一件事特別重要:環境設定檔(`.tool-versions`、`docker-compose.yml`、`devcontainer.json`)應該和程式碼放在同一個 git repo 裡,而且任何人改了服務版本,就應該同步更新這些檔案。這樣「更新環境」這件事才有機會被 code review 接住,而不是靠口頭通知。

### 3.3.2 Layer 2:讓回饋迴圈快

環境對齊只是第一步。接下來要問的是:**改了一行程式碼,多久後能看到效果?**

這個問題聽起來像「細節」,其實影響的是工程師一整天的工作節奏。我們都知道,人專注在一件事上很容易被打斷——而等待就是最常見的打斷來源。實踐中我們看到,如果改一行邏輯要等三十秒才能看到結果,工程師就會切去查信件或刷社群媒體;等回來時,原本的思路已經斷了。讓回饋迴圈快,是為了保護你自己的專注力。

**Hot Reload 與 Watch Mode**

幾乎所有主流框架都原生支援熱重載(Hot Reload),但不同語言生態的做法略有不同:

| 語言/框架 | 推薦工具 | 啟動指令範例 |
|---|---|---|
| Node.js 22+ | 內建 `--watch` 旗標 | `node --watch src/index.js` |
| Node.js(舊版) | `nodemon`(開源) | `nodemon src/index.js` |
| Python Flask | `flask run --reload` | — |
| Python FastAPI | `uvicorn main:app --reload` | — |
| Go | `air`(開源 live-reloader) | `air -c .air.toml` |
| Java Spring Boot | Spring Boot DevTools | `./mvnw spring-boot:run` |

以 ShopNova `order-service` 為例,在 `package.json` 裡加上一條 `dev` script:

```json
// package.json(Node.js 22.x + pnpm 9.x)
{
  "scripts": {
    "dev": "node --watch --env-file=.env src/server.js",
    "test:watch": "jest --watchAll --coverage=false"
  }
}
```

`node --watch` 在 Node.js 18.11 之後就內建了,不需要額外安裝套件。改動 `src/` 下任何 `.js` 檔案,服務會在約 1–2 秒內重啟——這個速度快到讓「存檔後立刻切到瀏覽器確認」變成自然而然的反射動作。

**只啟動你需要的子集**

一個大型微服務系統,在本機全部跑起來既慢又吃記憶體。遇到這種情況,一個實際可行的做法是:把你正在修的那個服務跑起來,其他服務用輕量的 stub 或 WireMock(開源 HTTP mock 伺服器,`wiremock/wiremock` Docker image)取代。

```bash
# 只跑 order-service 需要的基礎設施
docker compose --profile dev up -d

# 把 payment-service 用 WireMock 取代
docker run --rm -d -p 8080:8080 \
  -v $(pwd)/stubs:/home/wiremock \
  wiremock/wiremock:3.5.4
```

在 `stubs/` 資料夾裡放置 JSON mapping 檔,WireMock 就能模擬 payment-service 的回應——你不需要知道 payment-service 的 DB 架構,也不需要把它整個跑起來。

**讓測試自動跑到你願意頻繁檢驗**

如果跑一次單元測試要四十秒,你不會每次改完都跑。如果是四秒,你會——這是前提:測試本身要夠快,才有資格談下一步。除了讓測試本身更快,還要讓測試自動跑,這樣存檔後立刻有反饋,而不是要記得手動執行。`jest --watchAll` 或 Python 的 `pytest-watch`(開源)套件做的正是這件事:讓測試在每次存檔後自動執行,把測試從「想起來才跑」變成「寫完就自動跑」。

以 ShopNova 訂單服務的促銷計算邏輯為例,加了 watch mode 之後,修一個邊界條件的流程會變成:

```
修改 discount.js → 存檔 → 約 2 秒後 Jest 自動跑相關測試
→ 終端機顯示 PASS / FAIL → 立刻知道邊界條件有沒有被正確處理
```

這個「存檔→看結果」的迴圈,對程式碼品質的影響遠比你想像的大。它不只減少 bug,也讓「寫測試」這件事從「額外工作」變成「工作的一部分」——因為測試馬上給你回饋,而不是要你等。

### 3.3.3 Layer 3:讓環境問題在上線前被擋住

正因為 Layer 1 和 Layer 2 讓環境可以快速、一致地跑起來,Layer 3 要做的事才有意義——**用程式化的方式,在問題進入 PR 或 CI 之前,先在本機擋住它**。

這一層的核心想法是:**本機是你擋問題成本最低的地方**——比 PR review、比 CI、比 staging、更比線上便宜一百倍。問題越晚被發現,修復的成本越高:一個在本機就能擋掉的 lint 錯誤,如果等到 CI 才被抓住,中間多花了一次「推 PR → 等 CI → 看 log → 修改 → 再推」的完整循環。

做法一:**env-check 腳本**

寫一個簡單的腳本,在服務啟動前檢查必要的環境變數是否設定、資料庫是否可連通。以 ShopNova 為例:

```bash
#!/usr/bin/env bash
# scripts/check-env.sh — 啟動 order-service 前執行
set -euo pipefail

REQUIRED_VARS=(DATABASE_URL REDIS_URL NODE_ENV PORT)
MISSING=()

for VAR in "${REQUIRED_VARS[@]}"; do
  if [[ -z "${!VAR:-}" ]]; then
    MISSING+=("$VAR")
  fi
done

if [[ ${#MISSING[@]} -gt 0 ]]; then
  echo "❌  缺少以下環境變數,請確認 .env 檔案:" >&2
  for VAR in "${MISSING[@]}"; do
    echo "    - $VAR" >&2
  done
  echo "    參考:.env.example" >&2
  exit 1
fi

# 確認 DB 可連通(需要 pg_isready)
if ! pg_isready -d "$DATABASE_URL" -q; then
  echo "❌  PostgreSQL 連線失敗,請確認 docker-compose 是否有跑起來" >&2
  exit 1
fi

echo "✅  環境變數與資料庫連線確認完成"
```

這個腳本也應該放進 git,讓所有人共用。小雅那次的問題——環境變數只藏在 Confluence 某頁——如果有這個腳本,啟動的當下就會明確告訴你缺了什麼,而不是讓你跑了測試才在深處報錯。

做法二:**pre-commit hook**

在 git commit 之前自動跑 linter、型別檢查、或部分測試。工具推薦:

- **`pre-commit`**(Python 生態,但可管理任何語言的 hook):設定一份 `.pre-commit-config.yaml` 就能整合 ESLint、Prettier、mypy 等工具。
- **`husky` + `lint-staged`**(Node.js 生態):只對 staged 的檔案跑 lint,速度快。

以 ShopNova 的 Node.js 專案為例:

```json
// package.json — husky + lint-staged 設定
{
  "lint-staged": {
    "src/**/*.js": [
      "eslint --fix",
      "prettier --write"
    ],
    "src/**/*.test.js": [
      "jest --findRelatedTests --passWithNoTests"
    ]
  }
}
```

```yaml
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"
npx lint-staged
```

執行 `pnpm prepare` 安裝完 husky 之後,每次 `git commit` 前都會自動跑 lint 和相關測試——明顯的格式問題或語法錯誤在進 PR 之前就被攔截。

做法三:**本機跑 CI 子集**

主流 CI 工具(GitHub Actions、GitLab CI)都有辦法讓你在本機跑部分 job。**`act`**(開源工具,nektos/act)可以讓你在本機模擬 GitHub Actions 的執行環境:

```bash
# 安裝 act(macOS)
brew install act

# 模擬跑 GitHub Actions 的 test job
act -j test --secret-file .env
```

不一定要每次都跑完整流程,但「在推 PR 前,先跑一下 CI 腳本的最重要兩個步驟」,已經能擋掉大多數的「推完才發現 CI 爆了」的情況。

這三個做法從外往內收緊了防線:`env-check` 在啟動時擋;pre-commit hook 在 commit 時擋;`act` 在推 PR 前擋。三道關卡各自輕量,合起來幾乎不留死角。

---

現在你知道三層要做什麼了。接下來要面對的是:很多人在實施這些做法的過程中,會不經意地走進幾個常見的彎路。說出來,不是要提醒你別犯錯,而是讓你下次遇到時心裡有個底。

## 3.4 容易絆倒的地方

---

**絆倒處一:Docker Compose 設定越來越肥,啟動越來越慢,最後沒人用。**

這是 Layer 1 最常見的退化路徑。一開始只有一個 DB,後來加了 Redis、Kafka、Elasticsearch、Jaeger,啟動一次要五分鐘,大家就開始跳過它直接連 dev server。因為設定做了卻不被執行,就無法真正隔離環境變異——本機和 CI 的環境差異依然會導致「本機能跑、CI 不能跑」,當初要解決的問題還是沒被解決。環境設定有了,但沒人用,問題等於沒解。

更危險的是,這個退化完全是隱性的。沒有人在 commit message 或 PR discussion 裡記錄「我們決定不用 compose 了」,只是一個個小決定累積。等到真的有人想修復環境設定,卻發現已經有三個月的代碼和習慣是在沒有 compose 的狀態下寫的——代價比當初的維護成本高出許多。

> 修正方向:把 `docker-compose.yml` 拆成多個 profile(`docker-compose --profile dev`、`--profile infra-full`)。日常開發只跑輕量的 `dev` profile,需要完整測試才跑 `infra-full`。讓環境設定跟著使用情境走,而不是追求一份「什麼都有」的大全組合。同時在 PR checklist 裡加一條:「如果這個 PR 新增了外部依賴,是否已更新 compose 和對應的 profile?」

---

**絆倒處二:`.env` 檔案沒有範本,新人靠猜或靠問。**

`.env` 裡有 secret,當然不能 commit。但「不能 commit secret」不等於「不能 commit 環境變數的鍵名」。把配置規格和敏感值分開來看:`.env.example` 只列出鍵名和格式,值留空或填假值,commit 進 repo(這是 public 的規格);`.env` 的實際敏感值只留在本機,不 commit(這是 private 的值)。這樣新人看 `.env.example` 就知道需要哪些變數,而沒有洩漏任何 secret。

> 修正方向:把 `.env.example` 和 `env-check` 腳本放進 git。每次新增環境變數,連同更新這兩個檔案作為同一個 PR。讓「新增了一個環境變數」這件事在 review 的時候就被看見,不是靠人記。

---

**絆倒處三:Hot Reload 開著,但其實沒有真的在跑測試。**

Hot Reload 很方便,但它只能告訴你「服務起來了、API 回應了」。它不能告訴你「這個改動有沒有破壞邊界條件」。有些工程師習慣 hot reload 後直接手動點一點 Postman,就覺得「好了」。這沒有錯,但如果測試是存在的,那手動確認完應該還是要跑一下測試再 commit。

更微妙的問題是:watch mode 開著之後,有時候會讓人產生「環境一直是最新的」的錯覺。但 watch mode 的盲點是,它只能重啟應用層——重啟不到的地方,它就完全看不見。舉三個例子:(1) 資料庫 schema 升級了,watch mode 不會自動跑 migration,你的服務用舊的 schema 繼續跑,直到某個查詢報錯才會發現;(2) 測試用的 fixtures 過時了,和新的資料結構對不上,watch mode 也不會提醒你去更新;(3) 依賴服務的 stub 資料更新了,你本機的 mock 卻沒有同步,測試照樣是綠的,但那是在測一個已經不存在的 API contract。這三件事都不在應用層,所以 watch mode 重啟再快,也碰不到它們。

> 修正方向:把 watch mode 配置為同時跑「相關測試」,不只是重啟服務。Jest 的 `--watchAll` 模式、pytest 的 `pytest-watch` 套件,都可以讓測試跟著改動自動觸發。讓「存檔後,測試自動給你一個信號」成為工作流的一部分。

---

**絆倒處四:「之後再補文件」的 onboarding 永遠沒有之後。**

每一個「我下禮拜補文件」,幾乎都沒有補。不是壞習慣,而是記憶體清空了:等到有空的時候,你已經記不得自己上週在設定環境時踢到哪幾塊石頭。

> 修正方向:把寫 onboarding 腳本(或更新 README)列進 ticket 的 Definition of Done。如果做完一個功能或修完一個 bug 要求「對應的測試也要綠」,那「如果這個 PR 新增了環境依賴,就要更新 `.env.example` 和 README」也可以是同樣層次的要求。讓它變成流程,不要靠記憶力。

---

**絆倒處五:pre-commit hook 太重,工程師開始用 `--no-verify` 繞過去。**

Pre-commit hook 的設計初衷是讓「明顯的問題」在進 PR 前被擋住。但如果 hook 跑得太慢——例如跑完整個測試套件需要兩分鐘——工程師就會開始頻繁使用 `git commit --no-verify` 跳過它。一旦「繞過」變成習慣,這道防線就形同虛設。

更糟的是,`--no-verify` 的使用往往不會被記錄在任何地方。你的 CI 可能設定得很完善,但如果大家平時都繞過 hook,那 CI 的錯誤率就會飆高——因為問題應該在本機被擋住,卻每次都流到 CI 才被抓到。

> 修正方向:pre-commit hook 應該只跑「秒級」的檢查——lint、型別檢查、小型單元測試。完整的測試套件讓 CI 跑,不要塞進 hook。`lint-staged` 的設計就是只對 staged 的檔案跑,而不是跑整個 codebase,這樣速度通常可以控制在五秒以內。如果某個 hook 跑了超過十秒,先調查為什麼,而不是接受它。

---

**絆倒處六:版本管理工具和 CI 用的版本沒有連動,造成「本機 pass、CI 爆」。**

這個問題比聽起來更常見。你在 `.tool-versions` 裡寫了 `nodejs 22.4.0`,但 GitHub Actions 的 `workflow.yml` 裡用的是 `node-version: '20'`。本機測試全綠,推上去 CI 就跑出奇怪的錯誤——不是因為程式碼有問題,而是兩個地方用了不同版本的 Node。

這個分歧很容易在不知不覺間發生,特別是當 CI 設定和 `.tool-versions` 分別由不同的人維護時。

> 修正方向:讓 CI 讀 `.tool-versions`。GitHub Actions 的 `setup-node` action 支援 `node-version-file: '.tool-versions'` 這個參數,直接從 `.tool-versions` 裡讀版本號,而不是硬寫在 workflow 裡。這樣兩邊永遠從同一個來源取版本,更新版本只需要改一個地方。

```yaml
# .github/workflows/test.yml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version-file: '.tool-versions'   # 直接讀 .tool-versions
    cache: 'pnpm'
```

## 3.5 帶得走的工具 ⸺ 一頁式「開發環境基線確認單」

下面這份空白模板,可以作為你替任何一個後端服務建立環境設定的起點。它不是設定的全貌,而是**最容易被遺漏的那幾個維度**,用一張清單先把輪廓描出來。

```text
開發環境基線確認單 ⸺ {服務名稱}
更新日期:{YYYY-MM-DD}  維護者:{@github-handle}

── 1. 執行環境 ─────────────────────────────────────────
語言版本:       {例: Node.js 22.x}
套件管理器:      {例: pnpm 9.x}
版本管理工具:    {例: mise / .tool-versions / asdf}

── 2. 外部依賴(本機) ────────────────────────────────────
啟動方式:        {docker-compose up -d / 手動 / 其他}
compose profile:  {dev / test / full}
服務清單:         DB={版本}, Cache={版本}, MQ={版本 or N/A}

── 3. 環境變數 ─────────────────────────────────────────
範本位置:        .env.example
必填但非 secret:  {列出鍵名}
需從 secret mgr 取: {列出鍵名或說明取得方式}
env-check 腳本:   {scripts/check-env.sh 或 N/A}

── 4. 啟動流程 ─────────────────────────────────────────
第一次設定:      {一行或兩行 shell 命令}
日常啟動:        {一行命令}
確認服務健康:     {curl / healthcheck endpoint}

── 5. 快速回饋設定 ──────────────────────────────────────
Hot Reload:       {開啟方式或 N/A}
Watch Mode 測試:  {npm run test:watch / pytest-watch / N/A}
Pre-commit Hook:  {已安裝 / 需執行 pre-commit install / N/A}

── 6. 常見問題排除 ──────────────────────────────────────
Q: {最近一個月最常被問到的問題}
A: {解法}

Q: {第二個常見問題}
A: {解法}
```

為什麼只有這六欄?因為環境設定最容易犯的毛病是「想一次寫完所有細節」,結果文件越長越沒人讀。這六欄的目的是讓一個全新的人,在三十分鐘內能把服務跑起來。細節可以之後補,但輪廓要第一天就清楚。

### 3.5.1 範例:ShopNova 訂單管理服務的環境基線確認單

讓我們回到小雅那個週五下午。如果 ShopNova 的 repo 裡當時有這份確認單,那場兩個小時的環境折騰,大概二十分鐘就能結束了。下面是那個服務填好之後的版本:

```text
開發環境基線確認單 ⸺ order-service
更新日期: 2026-04-10  維護者: @wei-lin

── 1. 執行環境 ─────────────────────────────────────────
語言版本:       Node.js 22.4.x
套件管理器:      pnpm 9.12.x
<!-- 為什麼這欄:Node 版本不一致是「本機能跑 CI 不能跑」最常見的原因。
     寫清楚版本,搭配 .tool-versions,一次解決版本飄移的問題。 -->
版本管理工具:    mise (.tool-versions 已在 repo 根目錄)

── 2. 外部依賴(本機) ────────────────────────────────────
啟動方式:        docker-compose up -d --profile dev
<!-- 為什麼這欄:不加 --profile dev 會把 Kafka 也拉起來,啟動要五分鐘;
     加了 profile 只跑 PostgreSQL 17 + Redis 7,三十秒內好。 -->
compose profile:  dev(日常)/ full(跑 integration test 用)
服務清單:         DB=PostgreSQL 17.x, Cache=Redis 7.x, MQ=N/A(dev profile)

── 3. 環境變數 ─────────────────────────────────────────
範本位置:        .env.example(已 commit 進 repo)
必填但非 secret:  DATABASE_URL, REDIS_URL, NODE_ENV, PORT
<!-- 為什麼這欄:DATABASE_URL 是最容易忘記填的一個,填錯格式服務會靜默失敗
     而非明確報錯。格式是 postgresql://user:pass@localhost:5432/orders_dev -->
需從 secret mgr 取: PAYMENT_GATEWAY_KEY(見 1Password vault: shopnova-dev)
env-check 腳本:   scripts/check-env.sh(啟動前自動執行)

── 4. 啟動流程 ─────────────────────────────────────────
第一次設定:
  cp .env.example .env
  mise install
  docker-compose up -d --profile dev
  pnpm install && pnpm db:migrate
  bash scripts/check-env.sh
日常啟動:        docker-compose up -d --profile dev && pnpm dev
確認服務健康:     curl http://localhost:3000/health → 預期回傳 {"status":"ok"}

── 5. 快速回饋設定 ──────────────────────────────────────
Hot Reload:       pnpm dev(Node 22 --watch 模式,改動後 ~2s 重啟)
Watch Mode 測試:  pnpm test:watch(Jest watch,存檔後自動跑相關測試)
<!-- 為什麼這欄:測試 watch mode 是本章最值得投資的一個習慣;
     「存檔 → 三秒後看到測試結果」讓你幾乎不需要刻意記得跑測試。 -->
Pre-commit Hook:  執行 pnpm prepare 會自動安裝(husky + lint-staged)

── 6. 常見問題排除 ──────────────────────────────────────
Q: docker-compose up 回報 port 5432 已被占用
A: 本機可能有另一個 PostgreSQL 服務在跑。執行 lsof -i :5432 確認
   後,把本機的 PostgreSQL 停掉,或改 compose 的 port 為 5433。

Q: mise install 報錯「no version set for nodejs」
A: 確認 repo 根目錄有 .tool-versions 檔案,並執行 mise trust 信任
   這個目錄的設定。如果 .tool-versions 不存在,請聯繫 @wei-lin 補上。

Q: pnpm db:migrate 失敗,報錯 connection refused
A: 資料庫可能還沒起來。先執行 docker-compose ps 確認 db container 是
   Up 狀態;如果不是,執行 docker-compose up -d --profile dev 再等 10 秒。

Q: scripts/check-env.sh 報錯缺少 pg_isready
A: pg_isready 是 PostgreSQL client 工具。執行 brew install libpq &&
   brew link --force libpq 安裝(macOS)。Linux 用 apt install postgresql-client。
```

你看,這份確認單不長——填好的版本也只有四十行左右。但它能讓小雅在週五下午直接跑 `cp .env.example .env` 起手,然後按照啟動流程往下走,每一步都是確定性的,不需要靠猜或靠問人。**環境設定好不好,最直接的測試標準是:一個完全沒有脈絡的新人,能不能在三十分鐘內把服務跑起來。** 這份確認單就是守住這個標準的最小基礎。

常見問題排除欄位值得特別說明一下:好的 onboarding 文件不只告訴你「怎麼開始」,也告訴你「卡住了怎麼辦」。把過去一個月裡最常被問到的四個問題直接寫進去,讓新人不需要打擾任何人,自己就能找到答案——這是對他們時間最好的尊重。

## 3.6 本章回顧

讀完這一章,你應該已經能:

- [ ] 說明為什麼「可重現環境」不只是好習慣,而是讓「本機能跑」和「CI 能跑」對齊的基礎設施
- [ ] 用 docker-compose + profile + `.tool-versions`,把執行環境與外部依賴都版本化
- [ ] 撰寫 `env-check` 腳本,讓「缺少環境變數」在啟動時就被明確指出
- [ ] 設定 hot reload 和 watch mode 測試,讓回饋迴圈從「改完要記得跑」縮短到「存檔自動跑」
- [ ] 用輕量的 pre-commit hook 在推 PR 之前攔截明顯的問題,而不是讓 CI 做第一道關卡
- [ ] 填寫一份「開發環境基線確認單」,讓下一個接手的人不用靠猜、不用靠問

如果想先從一件事開始,我會建議 ⸺**建立 `.env.example` 並 commit 進 repo**,因為它做起來只要十分鐘,但它能一次解決「不知道需要哪些環境變數」這個折磨了無數人的問題;先做這一件,你已經讓下一個人的第一天好過很多了。

下一章,我們會繼續待在「把工作做紮實」這個主題上,但角度換到版本控制——不只是「怎麼用 git」,而是「分支策略怎麼選、commit 怎麼寫、PR 怎麼拆」,讓你的每一個改動都可以被追蹤、被還原、被理解。

## Cross-References

- **上一章**:[第 2 章｜讀懂一份陌生程式碼](./ch-02-reading-unfamiliar-code.md) ⸺ 讀懂程式碼的前提,是能把服務跑起來
- **下一章**:[第 4 章｜版本控制策略](./ch-04-version-control.md) ⸺ 環境對齊之後,下一步是讓改動可追蹤
- **強連結**:[第 15 章｜測試資料與測試環境](../part-03-testing/ch-15-test-data.md) ⸺ 本章談本機環境;第 15 章談測試環境的隔離與資料策略
- **強連結**:[第 16 章｜與 CI 整合的測試流水線](../part-03-testing/ch-16-ci-testing.md) ⸺ 本機的 pre-commit hook 和 CI 流水線是同一條防線的兩端
- **強連結**:[第 21 章｜CI/CD 流水線設計](../part-05-delivery/ch-21-cicd.md) ⸺ 本機環境盡量對齊 CI 環境,才能讓「本機能跑」等於「CI 能跑」
- **跨書連結**:[SA/SD Playbook](https://github.com/EddyKuo/sa-sd-playbook) ⸺ 容器化環境的設計決策屬於基礎設施架構高度,SA/SD 第 32–36 章有深入討論
