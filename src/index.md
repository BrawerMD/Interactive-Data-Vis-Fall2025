# WELCOME TO MAX'S WORLD

## I hope you are glad to be here

This is not my first time using GitHub, you know :)

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
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/1a/Data_visualization_process_v1.png/640px-Data_visualization_process_v1.png" alt="Data Visualization Process" width="500"/>

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
