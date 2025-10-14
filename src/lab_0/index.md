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

## Interactive Dashboard Element (JS Example)
<p>Change the heading color below:</p>

<input type="color" id="colorPicker" value="#ff6900">
<h3 id="demoText">Interactive Data Visualization Rocks!</h3>

<script>
  const picker = document.getElementById('colorPicker');
  const demoText = document.getElementById('demoText');
  picker.addEventListener('input', () => {
    demoText.style.color = picker.value;
  });
</script>

---

## Observable Input Toast Example
<div id="observable-input"></div>
<div id="toast"></div>

<script type="module">
  import * as Inputs from "https://cdn.jsdelivr.net/npm/@observablehq/inputs@0.10/dist/inputs.js";

  const container = document.getElementById("observable-input");
  const toast = document.getElementById("toast");

  const input = Inputs.text({
    label: "Say something:",
    placeholder: "Type and press Enter",
    width: "400px"
  });

  container.appendChild(input);

  input.addEventListener("change", () => {
    showToast(`You typed: ${input.value}`);
    input.value = "";
  });

  function showToast(msg) {
    toast.textContent = msg;
    toast.style.visibility = "visible";
    toast.style.opacity = "1";
    toast.style.position = "fixed";
    toast.style.bottom = "30px";
    toast.style.left = "50%";
    toast.style.transform = "translateX(-50%)";
    toast.style.background = "#ff6900";
    toast.style.color = "#fff";
    toast.style.padding = "10px 20px";
    toast.style.borderRadius = "4px";
    setTimeout(() => {
      toast.style.opacity = "0";
      setTimeout(() => (toast.style.visibility = "hidden"), 500);
    }, 2000);
  }
</script>
