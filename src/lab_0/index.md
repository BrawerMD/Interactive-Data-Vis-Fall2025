# WELCOME TO MAX'S WORLD

## I hope you are glad to be here

This is not my first time using GitHub! I miss this site.

---

## Course Info Table (HTML Example)
<table>
  <tr>
    <th>Week</th>
    <th>Topic</th>
    <th>Lab</th>
  </tr>
  <tr>
    <td>1</td>
    <td>Intro & Setup</td>
    <td>Lab 0</td>
  </tr>
  <tr>
    <td>2</td>
    <td>D3 Data Binding</td>
    <td>Lab 1</td>
  </tr>
  <tr>
    <td>3</td>
    <td>Scales & Axes</td>
    <td>Lab 2</td>
  </tr>
</table>

---

## Learning Highlights
<ul>
  <li>Understanding data visualization principles</li>
  <li>Hands-on practice with D3.js</li>
  <li>Building interactive charts and dashboards</li>
</ul>

---

## Visual Example
<img src="https://s3-eu-central-1.amazonaws.com/euc-cdn.freshdesk.com/data/helpdesk/attachments/production/80125162144/original/LYc_NmbLV_lLf-ZQMumImQIfOM1yt5ZhQQ.png?1658772841" alt="Data Visualization Is Cool" width="500"/>

---

## Color Picker
<p>Change the heading color below:</p>

## 🎨 Interactive Color Picker

```js
const textColor = view(Inputs.color({
  label: "Text Color",
  value: "#ff6900"
}))
```
```html
Data Viz is <span style="color:${textColor}">colorful</span>!
```
---
# Observable Table Example

```js
Inputs.table([
  ["ProductID", "Category", "InStock"],
  [435, "Apparel", true],
  [87, "Electronics", false],
  [912, "Home Goods", false],
  [330, "Toys", true],
  [562, "Books", true]
])
```
# Population Explorer

```js
const year = view(Inputs.range([1800, 2020], {
  label: "Select Year",
  step: 1,
  value: 2000
}));
```

The population data for your selected year is **${year}**.

</script>