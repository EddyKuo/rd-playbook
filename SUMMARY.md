# 《RD Playbook ⸺ 研發工程師決策手冊 2026》

[序章 ⸺ 當 AI 會寫程式之後,工程師還剩下什麼](front-matter/00-preface.md)

---

## Part I｜研發基礎

- [篇導讀](part-01-foundations/00-overview.md)
- [第 1 章｜為什麼工程實作需要決策框架](part-01-foundations/ch-01-why-engineering-decisions.md)
- [第 2 章｜讀懂一份陌生程式碼](part-01-foundations/ch-02-reading-unfamiliar-code.md)
- [第 3 章｜開發環境與本機工作流](part-01-foundations/ch-03-dev-environment.md)
- [第 4 章｜版本控制策略](part-01-foundations/ch-04-version-control.md)

## Part II｜程式碼工藝

- [篇導讀](part-02-craft/00-overview.md)
- [第 5 章｜可讀性:為下一個人而寫](part-02-craft/ch-05-readability.md)
- [第 6 章｜命名、抽象與邊界](part-02-craft/ch-06-naming-abstraction.md)
- [第 7 章｜控制複雜度](part-02-craft/ch-07-complexity.md)
- [第 8 章｜重構的時機與安全網](part-02-craft/ch-08-refactoring.md)
- [第 9 章｜程式碼層的技術債](part-02-craft/ch-09-tech-debt.md)

## Part III｜測試工程

- [篇導讀](part-03-testing/00-overview.md)
- [第 10 章｜可測試的程式碼設計](part-03-testing/ch-10-testable-code.md)
- [第 11 章｜單元測試與 TDD 的落地](part-03-testing/ch-11-unit-tdd.md)
- [第 12 章｜契約測試與整合測試](part-03-testing/ch-12-contract-integration.md)
- [第 13 章｜測試替身的取捨](part-03-testing/ch-13-test-doubles.md)
- [第 14 章｜測試資料與測試環境](part-03-testing/ch-14-test-data.md)
- [第 15 章｜與 CI 整合的測試流水線](part-03-testing/ch-15-ci-testing.md)

## Part IV｜協作與審查

- [篇導讀](part-04-collaboration/00-overview.md)
- [第 16 章｜Code Review:看什麼、怎麼給回饋](part-04-collaboration/ch-16-code-review.md)
- [第 17 章｜Pull Request 的拆分與描述](part-04-collaboration/ch-17-pull-request.md)
- [第 18 章｜結對與群體程式設計的時機](part-04-collaboration/ch-18-pairing.md)
- [第 19 章｜跨團隊的介面契約與相依](part-04-collaboration/ch-19-interface-contracts.md)

## Part V｜交付工程

- [篇導讀](part-05-delivery/00-overview.md)
- [第 20 章｜CI/CD 流水線設計](part-05-delivery/ch-20-cicd.md)
- [第 21 章｜Feature Flag 與漸進式發布](part-05-delivery/ch-21-feature-flags.md)
- [第 22 章｜藍綠與金絲雀部署](part-05-delivery/ch-22-blue-green-canary.md)
- [第 23 章｜回滾與前向修復決策](part-05-delivery/ch-23-rollback.md)
- [第 24 章｜資料庫遷移與零停機變更](part-05-delivery/ch-24-db-migration.md)

## Part VI｜維運工程

- [篇導讀](part-06-operations/00-overview.md)
- [第 25 章｜可觀測性落地](part-06-operations/ch-25-observability.md)
- [第 26 章｜從告警到根因:生產環境除錯](part-06-operations/ch-26-alert-to-rootcause.md)
- [第 27 章｜向下穿透抽象層](part-06-operations/ch-27-penetrating-abstraction.md)
- [第 28 章｜On-call 與事故處理](part-06-operations/ch-28-on-call.md)
- [第 29 章｜SLO 與錯誤預算的實作面](part-06-operations/ch-29-slo.md)
- [第 30 章｜災難復原演練](part-06-operations/ch-30-disaster-recovery.md)

## Part VII｜效能與可靠性工程

- [篇導讀](part-07-performance/00-overview.md)
- [第 31 章｜效能量測先於優化](part-07-performance/ch-31-measure-first.md)
- [第 32 章｜瓶頸定位](part-07-performance/ch-32-bottleneck.md)
- [第 33 章｜快取的層次與失效策略](part-07-performance/ch-33-caching.md)
- [第 34 章｜並發、非同步與背壓](part-07-performance/ch-34-concurrency.md)
- [第 35 章｜容量規劃與壓力測試](part-07-performance/ch-35-capacity.md)

## Part VIII｜AI 時代研發

- [篇導讀](part-08-ai-era/00-overview.md)
- [第 36 章｜AI 輔助編碼的工作流重塑](part-08-ai-era/ch-36-ai-assisted-coding.md)
- [第 37 章｜審查 AI 生成的程式碼](part-08-ai-era/ch-37-reviewing-ai-code.md)
- [第 38 章｜為 AI 生成碼補測試與防護](part-08-ai-era/ch-38-testing-ai-code.md)
- [第 39 章｜Agentic 開發](part-08-ai-era/ch-39-agentic-dev.md)
- [第 40 章｜Prompt 與 context 作為工程產物](part-08-ai-era/ch-40-prompt-as-artifact.md)
- [第 41 章｜AI 時代的工程師心智與責任界線](part-08-ai-era/ch-41-engineer-mindset.md)

## Part IX｜綜合演練

- [篇導讀](part-09-synthesis/00-overview.md)
- [第 42 章｜從設計到上線:一個完整功能的實作全紀錄](part-09-synthesis/ch-42-feature-end-to-end.md)
- [第 43 章｜一次生產事故的完整復盤](part-09-synthesis/ch-43-incident-postmortem.md)
- [第 44 章｜接手 legacy 系統的 90 天計畫](part-09-synthesis/ch-44-legacy-90-days.md)
- [第 45 章｜判斷力的養成:當階梯被 AI 抽掉](part-09-synthesis/ch-45-cultivating-judgment.md)
- [第 46 章｜研發者的成長路徑](part-09-synthesis/ch-46-growth-path.md)
