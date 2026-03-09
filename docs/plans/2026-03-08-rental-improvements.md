# レンタル改善・UI修正 実装プラン

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 期間計算修正・カレンダー入力・フルネームハンコ・スケジュール原価の5項目を修正する

**Architecture:** indexファイル（フロントエンド）とCodeファイル（GAS）の2ファイルを修正。GASはgetStaffInfoに氏名列を追加し、フロントへ渡す。

**Tech Stack:** React 18 (Babel), GAS, Tailwind CSS

---

## Task 1: 期間計算ロジック修正（index line 452）

**Files:** Modify `index`

**変更内容:**
```
現在:
return (end.getFullYear() - start.getFullYear()) * 12 + (end.getMonth() - start.getMonth());

修正後:
const monthDiff = (end.getFullYear() - start.getFullYear()) * 12 + (end.getMonth() - start.getMonth());
return monthDiff + (end.getDate() >= start.getDate() ? 1 : 0);
```

**検証:**
- 2026/03/01〜2026/05/31 → 3ヶ月
- 2026/03/11〜2026/05/10 → 2ヶ月
- 2026/09/20〜2026/12/19 → 3ヶ月

---

## Task 2: generateScheduleのアラートメッセージ修正（index line 521）

**Files:** Modify `index`

**変更内容:** エラーメッセージの説明を正しい計算方式に更新

---

## Task 3: generateScheduleの原価計算修正（index line 528）

**Files:** Modify `index`

**変更内容:**
```
// 追加
const monthlyCostTotal = items.filter(i => i.productCode !== '13').reduce((sum, i) => sum + (Number(i.costAmount) || 0), 0);
const shippingCostTotal = items.filter(i => i.productCode === '13').reduce((sum, i) => sum + (Number(i.costAmount) || 0), 0);

// costReferenceを修正
costReference: isFirst ? monthlyCostTotal + shippingCostTotal : monthlyCostTotal,
```

---

## Task 4: GAS getStaffInfoに氏名列追加（Code line 46-82）

**Files:** Modify `Code`

**変更内容:**
- `fullNameIdx = headers.indexOf("氏名")` を追加
- resultに `fullName: fullNameIdx !== -1 ? String(data[i][fullNameIdx]).trim() : ""` を追加
- fallback: `{ role: "", title: "", name: "", fullName: "" }`

---

## Task 5: GAS doGetにuserFullNameを追加（Code line 35-38）

**Files:** Modify `Code`

**変更内容:**
```
template.userFullName = staffInfo.fullName || staffInfo.name || paramUserName || "担当者";
```

---

## Task 6: indexのgas-dataにdata-user-full-name属性追加（index line 44）

**Files:** Modify `index`

**変更内容:**
```html
data-user-full-name="<?= userFullName ?>"
```

---

## Task 7: CONFIGにuserFullNameを追加（index line 79）

**Files:** Modify `index`

**変更内容:**
```
userFullName: gasDataEl?.dataset.userFullName || gasDataEl?.dataset.userName || "担当者"
```

---

## Task 8: handleApproveActionをフルネームに変更（index line 621-622）

**Files:** Modify `index`

**変更内容:**
```
現在: const approverName = (CONFIG.userName && CONFIG.userName !== '担当者') ? CONFIG.userName : stampLabel;
修正後: const approverName = (CONFIG.userFullName && CONFIG.userFullName !== '担当者') ? CONFIG.userFullName : ((CONFIG.userName && CONFIG.userName !== '担当者') ? CONFIG.userName : stampLabel);
```

---

## Task 9: Stampコンポーネントの名前表示修正（index line 694）

**Files:** Modify `index`

**変更内容:**
```
現在: <span className="text-[9px] font-bold leading-tight text-center break-all">{info.name}</span>

修正後:
<span className="text-[8px] font-bold leading-tight text-center whitespace-pre-line">
  {info.name.replace(/[\s　]+/, '\n')}
</span>
```

---

## Task 10: 出荷日・納期をtype="date"に変更（index 出荷行レンダリング部分）

**Files:** Modify `index`

**変更内容:** 出荷日・納期の2つのinputを以下に変更:
```jsx
// 出荷日
<input type="date"
  className={`w-full text-center border-b border-gray-300 outline-none text-sm bg-transparent ${errors[`shipDate${n}`] ? 'input-error' : ''}`}
  value={(header[`shipDate${n}`] || '').replace(/\//g, '-')}
  onChange={e => updateHeader(`shipDate${n}`, e.target.value.replace(/-/g, '/'))}
  onBlur={...既存のバリデーション}
  disabled={isLocked}
/>
// 納期
<input type="date"
  className="w-full text-center border-b border-gray-300 outline-none text-sm bg-transparent"
  value={(header[`deliveryDate${n}`] || '').replace(/\//g, '-')}
  onChange={e => updateHeader(`deliveryDate${n}`, e.target.value.replace(/-/g, '/'))}
  disabled={isLocked}
/>
```

---

## Task 11: コミット＆clasp push

```bash
git add Code index
git commit -m "feat: 期間計算修正/カレンダー入力/フルネームハンコ/スケジュール原価"
cp Code Code.gs && cp index index.html && cp appsscript appsscript.json
clasp push --force
rm -f Code.gs index.html appsscript.json
```

---

## Task 12: コードレビュー

superpowers:requesting-code-review を実行して全変更を検証する
