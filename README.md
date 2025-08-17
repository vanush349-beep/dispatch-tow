<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>EDL Towing Service</title>
<style>
  body { font-family: Arial, sans-serif; background:#f8f9fa; display:flex; flex-direction:column; align-items:center; padding:20px; }
  header { text-align:center; margin-bottom:30px; }
  header img { width:120px; margin-bottom:10px; }
  header h1 { margin:0; color:#007bff; font-size:2em; }
  header p { color:#555; margin-top:5px; }
  form { background:#fff; padding:20px; border-radius:10px; box-shadow:0 4px 10px rgba(0,0,0,0.1); max-width:400px; width:100%; display:flex; flex-direction:column; margin-bottom:20px; }
  input { margin-bottom:10px; padding:10px; border-radius:5px; border:1px solid #ccc; font-size:16px; width:100%; box-sizing:border-box; }
  button { padding:10px; font-size:16px; border:none; border-radius:5px; background:#007bff; color:white; cursor:pointer; }
  button:hover { background:#0056b3; }
  .section { background:#fff; padding:20px; border-radius:10px; box-shadow:0 4px 10px rgba(0,0,0,0.1); max-width:400px; width:100%; text-align:center; margin-bottom:20px; }
  .driver-item { display:flex; justify-content:space-between; align-items:center; margin-bottom:5px; padding:5px; border-bottom:1px solid #ddd; }
  .driver-item button { background:#dc3545; padding:5px 10px; border:none; border-radius:3px; color:white; cursor:pointer; }
  .driver-item button:hover { background:#c82333; }
  .driver-item a { background:#28a745; padding:5px 10px; border-radius:3px; color:white; text-decoration:none; margin-right:5px; }
  .driver-item a:hover { background:#218838; }
</style>
</head>
<body>

<header>
  <img src="https://i.imgur.com/6r0B6VY.png" alt="EDL Towing Logo">
  <h1>EDL Towing Service</h1>
  <p>Fast • Reliable • Trustworthy</p>
</header>

<!-- Tow Request Form -->
<form id="towForm">
  <input type="text" id="name" placeholder="Your Name" required>
  <input type="text" id="details" placeholder="Vehicle Details / Location" required>
  <button type="submit">Request Tow</button>
</form>

<!-- Driver Management Section -->
<div class="section">
  <h2>Manage Drivers</h2>
  <form id="addDriverForm">
    <input type="text" id="driverName" placeholder="Driver Name" required>
    <input type="number" id="driverLat" placeholder="Latitude" step="any" required>
    <input type="number" id="driverLng" placeholder="Longitude" step="any" required>
    <input type="text" id="driverPhone" placeholder="Phone Number (optional)">
    <button type="submit">Add Driver</button>
  </form>
  <div id="driversList"></div>
</div>

<script>
let drivers = [];

// Calculate distance between two points
function distance(lat1, lon1, lat2, lon2) {
  const R = 6371;
  const dLat = (lat2-lat1)*Math.PI/180;
  const dLon = (lon2-lon1)*Math.PI/180;
  const a = Math.sin(dLat/2)**2 + Math.cos(lat1*Math.PI/180)*Math.cos(lat2*Math.PI/180)*Math.sin(dLon/2)**2;
  const c = 2*Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
  return R*c;
}

// Tow request form
document.getElementById("towForm").addEventListener("submit", function(e){
  e.preventDefault();
  if(drivers.length === 0){ alert("No drivers available."); return; }
  if(navigator.geolocation){
    navigator.geolocation.getCurrentPosition(position => {
      const userLat = position.coords.latitude;
      const userLng = position.coords.longitude;
      let closest = drivers[0];
      let minDist = distance(userLat, userLng, closest.lat, closest.lng);
      drivers.forEach(driver => {
        const d = distance(userLat, userLng, driver.lat, driver.lng);
        if(d < minDist){ minDist = d; closest = driver; }
      });
      const name = encodeURIComponent(document.getElementById("name").value);
      const details = encodeURIComponent(document.getElementById("details").value);
      const driverName = encodeURIComponent(closest.name);
      const smsUrl = `https://api.callmebot.com/sms.php?phone=+19636660001&text=Tow request from ${name}. Vehicle/Location: ${details}. Assigned Driver: ${driverName}`;
      window.open(smsUrl, "_blank");
    });
  } else { alert("Geolocation not supported."); }
});

// Add driver
const addDriverForm = document.getElementById("addDriverForm");
addDriverForm.addEventListener("submit", function(e){
  e.preventDefault();
  const name = document.getElementById("driverName").value;
  const lat = parseFloat(document.getElementById("driverLat").value);
  const lng = parseFloat(document.getElementById("driverLng").value);
  const phone = document.getElementById("driverPhone").value;
  drivers.push({ name, lat, lng, phone });
  updateDriversList();
  addDriverForm.reset();
});

// Update driver list
function updateDriversList(){
  const list = document.getElementById("driversList");
  list.innerHTML = "";
  drivers.forEach((driver, index) => {
    const div = document.createElement("div");
    div.className = "driver-item";
    const wazeLink = `https://waze.com/ul?ll=${driver.lat},${driver.lng}&navigate=yes`;
    div.innerHTML = `${driver.name} 
      <a href="${wazeLink}" target="_blank">View on Waze</a>
      <button onclick="removeDriver(${index})">Remove</button>`;
    list.appendChild(div);
  });
}

// Remove driver
function removeDriver(index){
  drivers.splice(index, 1);
  updateDriversList();
}
</script>

</body>
</html>