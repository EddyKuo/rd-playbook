# Conventions — RD Playbook 章節格式與 cross-ref 規範

> 補充 `voice-guide.md` 的格式細節。所有章節一律遵守。

---

## 1. 檔案命名

- **主章**:`part-{NN}-{slug}/ch-{NN}-{kebab-slug}.md`
- **Part 導讀**:`part-{NN}-{slug}/00-overview.md`
- **Front Matter**:`front-matter/{NN}-{kebab-slug}.md`

## 2. Frontmatter(YAML,章節必填)

```yaml
---
chapter: 1
part: I                     # I–IX
title: 章節完整標題
slug: kebab-case-slug
agent: RD                   # 本書主筆角色預設 RD
domain_case: CASE-XXX-NNN   # 主案例 ID(須已註冊於 _refs/case-registry.yaml)
status: draft               # draft / review / approved
word_count_target: 5500
---
```

## 3. 章首 Cross-Ref Block

緊接 H1 / 副標之後,以 blockquote 呈現:

```markdown
> **前置閱讀**:[Ch X 標題](../part-NN-slug/ch-XX-slug.md)
> **下游章節**:[Ch Y 標題](../part-NN-slug/ch-YY-slug.md)
```

## 4. 章末 Cross-References Section

```markdown
## Cross-References

- **下一章**:[Ch X 標題](./ch-XX-slug.md) ⸺ 一句話描述
- **強連結**:[Ch Y 標題](../part-NN-slug/ch-YY-slug.md)
- **跨書連結**:[SA/SD Ch Z](https://github.com/EddyKuo/sa-sd-playbook) / [PM Playbook](https://github.com/EddyKuo/pm-playbook)
```

> 跨書連結:RD 與 SA/SD、PM、QA 互通。引用他書章節時,以書名 + 章名標示。

## 5. 章節內標題層級

- `H1` 章節大標(只出現一次),其後 `## ⸺ {副標}` 作為副標
- `H2`(`## X.Y 段落`):六大段(共感 / 真問題 / 判斷 / 絆倒處 / 工具 / 回顧)
- `H3`(`### X.Y.Z 子段`)
- `H4` 偶用於極細分

## 6. Mermaid 樣式

```
classDef hot  fill:#fee,stroke:#c33     # 危險 / 反模式 / 痛點
classDef cold fill:#eef,stroke:#36c     # 中性 / 起始狀態
classDef goal fill:#efe,stroke:#3a3     # 目標 / 推薦做法
```

## 7. 案例 ID 配置

| 類型 | 命名 | 範圍 |
|---|---|---|
| Cases | `CASE-{DOMAIN}-NNN` | DOMAIN 為 FIN/ECM/HCR/SAS/ENR |

新案例先註冊於 `_refs/case-registry.yaml` 才能在內文以 ID 引用。

## 8. §X.5 交付清單格式

每章 §X.5 必須是「**空白模板 + 填好範例**」雙段結構:

```text
## X.5 帶得走的工具 ⸺ 一頁式 {Artifact 名稱}
   ↓ 空白模板 code block(用 {placeholder})
   ↓ 「為什麼是一頁 / 為什麼有這些欄位」prose
### X.5.1 範例:{案例名}
   ↓ 1–2 句 intro,扣回本章故事
   ↓ 填好的範例 code block(關鍵欄位以 <!-- 為什麼這欄:… --> 註解)
   ↓ 1–2 句 punchline 收尾(REASSURING 語氣)
```

- 範例須使用 frontmatter `domain_case:` 指定、且章內已鋪陳故事的案例。
- 欄位註解只挑「沒寫會吃虧」的 3–5 個關鍵欄位,語氣溫暖(講後果與設計意圖,不命令)。
- 範例 code block 主體 ≤ 60 行。

## 9. 章末 Recap 格式

```markdown
## X.6 本章回顧

讀完這一章,你應該已經能:

- [ ] {動作 1}
- [ ] {動作 2}
- [ ] {動作 3}

如果想先從一件事開始,我會建議 ⸺{動作 X},因為 {溫暖的理由}。
```

## 10. PROPOSED-REFS(寫作時新增案例用)

新章節若引入新案例,不直接編輯 registry,於章末附 HTML comment:

```html
<!-- PROPOSED-REFS
cases:
  - id: CASE-XXX-NNN
    title: "..."
    domain: ...
    chapters: [ch-NN]
    summary: |
      ...
-->
```

由維護者在審校通過後合併到 `_refs/case-registry.yaml`,再刪除此區塊。
