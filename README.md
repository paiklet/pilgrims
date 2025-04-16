<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0"> <!-- مهم لدعم الجوال -->
    <title>إدارة الفنادق - الحجاج</title>
    <style>
        body { font-family: 'Segoe UI', sans-serif; margin: 20px; background: #f5f5f5; direction: rtl; }
        h2 { color: #333; }
        .section { margin-bottom: 20px; background: white; padding: 15px; border-radius: 8px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
        input[type="number"], input[type="text"], select { padding: 5px; margin: 5px; width: 100%; max-width: 300px; }
        button { padding: 7px 15px; margin: 5px 2px; background: #4CAF50; color: white; border: none; cursor: pointer; border-radius: 4px; width: 100%; max-width: 200px; }
        .room { margin: 10px 0; padding: 10px; background: #e9f5e9; border-radius: 6px; display: inline-block; width: 100%; }
        .bed { margin: 5px 0; }
        #searchResult { margin-top: 10px; font-weight: bold; color: #2b2b2b; }
        .controls { display: inline-block; margin-right: 10px; }
        .hotelName { color: blue; font-size: 24px; font-weight: bold; margin-bottom: 20px; }
        .roomContainer { display: flex; flex-direction: column; gap: 10px; }
        .header { text-align: center; margin-bottom: 20px; }
        
        /* Responsive Design */
        @media screen and (max-width: 768px) {
            body { margin: 10px; }
            .section { padding: 10px; }
            button { width: 100%; max-width: 100%; }
            .room { width: 100%; }
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>إدارة الفنادق - الحجاج</h1>
        <div class="hotelName" id="hotelNameDisplay">اسم الفندق</div>
    </div>

    <!-- New Section for Adding Hotels -->
    <div class="section">
        <h2>إضافة عدد الفنادق</h2>
        <input type="number" id="hotelCount" min="1" placeholder="عدد الفنادق">
        <button onclick="addHotels()">إضافة الفنادق</button>
    </div>

    <div class="section" id="hotelSelectorSection" style="display:none;">
        <h2>اختيار الفندق</h2>
        <select id="hotelSelector" onchange="selectHotel()">
            <option disabled selected>اختر الفندق</option>
        </select>
    </div>

    <div class="section">
        <h2>إضافة عدد الأدوار للفندق المحدد</h2>
        <input type="number" id="floorCount" min="1" placeholder="عدد الأدوار">
        <button onclick="addFloors()">إضافة</button>
    </div>

    <div class="section">
        <h2>اختيار الطابق للفندق المحدد</h2>
        <select id="floorSelector" onchange="selectFloor()">
            <option disabled selected>اختر الطابق</option>
        </select>
    </div>

    <div class="section" id="floorConfig" style="display:none;">
        <h2>إعداد الغرف للطابق</h2>
        <label>عدد الغرف: <input type="number" id="roomCount" min="1"></label>
        <label>عدد الأسرة في كل غرفة: <input type="number" id="bedsPerRoom" min="1"></label><br>
        <button onclick="generateRooms()">تأكيد</button>
    </div>

    <div class="section">
        <h2>توزيع الحجاج</h2>
        <div id="roomDisplay"></div>
        <button onclick="window.print()">🖨️ طباعة / تصدير PDF</button>
        <button onclick="deleteAllData()">حذف البيانات</button>
    </div>

    <div class="section">
        <h2>بحث عن حاج</h2>
        <input type="text" id="searchName" placeholder="اكتب الاسم">
        <button onclick="searchPerson()">بحث</button>
        <div id="searchResult"></div>
    </div>

    <div class="section">
        <h2>عدد الأسرة الفارغة في الطابق الحالي</h2>
        <button onclick="countEmptyBeds()">حساب عدد الأسرة الفارغة في الطابق الحالي</button>
        <div id="emptyBedsCount"></div>
    </div>

    <div class="section">
        <h2>جميع الأسرة الفارغة في الفندق</h2>
        <button onclick="countAllEmptyBeds()">حساب جميع الأسرة الفارغة</button>
        <div id="allEmptyBedsCount"></div>
    </div>

    <script>
        let hotels = {};
        let currentHotel = null;
        let floorCount = 0;

        function addHotels() {
            const hotelCount = parseInt(document.getElementById("hotelCount").value);
            if (!hotelCount || hotelCount <= 0) return alert("أدخل عدد الفنادق بشكل صحيح.");
            hotels = {}; // Reset hotels data
            for (let i = 1; i <= hotelCount; i++) {
                hotels[i] = { floors: [] };
            }
            updateHotelSelector();
            document.getElementById("hotelSelectorSection").style.display = "block";
        }

        function updateHotelSelector() {
            const selector = document.getElementById("hotelSelector");
            selector.innerHTML = '<option disabled selected>اختر الفندق</option>';
            for (let i = 1; i <= Object.keys(hotels).length; i++) {
                const opt = document.createElement("option");
                opt.value = i;
                opt.textContent = `الفندق ${i}`;
                selector.appendChild(opt);
            }
        }

        function selectHotel() {
            const hotel = document.getElementById("hotelSelector").value;
            currentHotel = hotel;
            document.getElementById("hotelNameDisplay").textContent = `فندق ${hotel}`;
            updateFloorSelector();
        }

        function addFloors() {
            floorCount = parseInt(document.getElementById("floorCount").value);
            if (!floorCount || floorCount <= 0) return alert("أدخل عدد الأدوار بشكل صحيح.");
            for (let i = 0; i < floorCount; i++) {
                hotels[currentHotel].floors.push({ rooms: [] });
            }
            updateFloorSelector();
            document.getElementById("floorConfig").style.display = "block";
        }

        function updateFloorSelector() {
            const selector = document.getElementById("floorSelector");
            selector.innerHTML = '<option disabled selected>اختر الطابق</option>';
            for (let i = 0; i < hotels[currentHotel].floors.length; i++) {
                const opt = document.createElement("option");
                opt.value = i;
                opt.textContent = `الطابق ${i + 1}`;
                selector.appendChild(opt);
            }
        }

        function selectFloor() {
            const floor = document.getElementById("floorSelector").value;
            displayRooms(floor);
        }

        function generateRooms() {
            const count = parseInt(document.getElementById("roomCount").value);
            const beds = parseInt(document.getElementById("bedsPerRoom").value);
            if (!count || !beds) return alert("أدخل عدد الغرف وعدد الأسرة.");
            const selectedFloor = document.getElementById("floorSelector").value;
            const rooms = Array.from({ length: count }, (_, i) => ({
                roomNumber: i + 1,
                beds: Array.from({ length: beds }, () => "")
            }));
            hotels[currentHotel].floors[selectedFloor].rooms = rooms;
            displayRooms(selectedFloor);
        }

        function displayRooms(floorIdx) {
            const display = document.getElementById("roomDisplay");
            display.innerHTML = "";
            const floor = hotels[currentHotel].floors[floorIdx];
            if (!floor || !floor.rooms) return;
            floor.rooms.forEach((room, i) => {
                let html = `<div class="room">غرفة ${room.roomNumber}<br>`;
                room.beds.forEach((name, j) => {
                    html += `
                        <div class="bed">
                            سرير ${j + 1}: 
                            <input type="text" value="${name}" onchange="saveName(${floorIdx}, ${i}, ${j}, this.value)">
                            <span class="controls">
                                <button onclick="transferGuest(${floorIdx}, ${i}, ${j})">نقل</button>
                                <button onclick="deleteGuest(${floorIdx}, ${i}, ${j})" style="background:#d9534f">حذف</button>
                            </span>
                        </div>`;
                });
                html += `</div>`;
                display.innerHTML += html;
            });
        }

        function saveName(floorIdx, roomIdx, bedIdx, name) {
            hotels[currentHotel].floors[floorIdx].rooms[roomIdx].beds[bedIdx] = name;
        }

        function transferGuest(floorIdx, roomIdx, bedIdx) {
            const guestName = hotels[currentHotel].floors[floorIdx].rooms[roomIdx].beds[bedIdx];
            const newFloor = prompt("اختر الطابق الجديد:");
            const newRoom = prompt("اختر رقم الغرفة:");
            const newBed = prompt("اختر رقم السرير:");

            if (!guestName || !newFloor || !newRoom || !newBed) return alert("من فضلك تأكد من إدخال جميع المعلومات.");
            
            const targetFloor = hotels[currentHotel].floors[newFloor - 1];
            if (!targetFloor || !targetFloor.rooms[newRoom - 1]) return alert("الطابق أو الغرفة المحددة غير موجودة.");

            const targetRoom = targetFloor.rooms[newRoom - 1];
            if (targetRoom.beds[newBed - 1] !== "") return alert("السرير المحدد ليس فارغًا.");

            targetRoom.beds[newBed - 1] = guestName;
            hotels[currentHotel].floors[floorIdx].rooms[roomIdx].beds[bedIdx] = "";
            displayRooms(floorIdx);
        }

        function deleteGuest(floorIdx, roomIdx, bedIdx) {
            hotels[currentHotel].floors[floorIdx].rooms[roomIdx].beds[bedIdx] = "";
            displayRooms(floorIdx);
        }

        function countEmptyBeds() {
            let emptyBedsCount = 0;
            const selectedFloor = document.getElementById("floorSelector").value;
            const floor = hotels[currentHotel].floors[selectedFloor];
            let roomsWithEmptyBeds = [];
            floor.rooms.forEach((room, idx) => {
                const emptyBedsInRoom = room.beds.filter(bed => bed === "").length;
                if (emptyBedsInRoom > 0) {
                    roomsWithEmptyBeds.push(`غرفة ${room.roomNumber} تحتوي على ${emptyBedsInRoom} أسرة فارغة`);
                    emptyBedsCount += emptyBedsInRoom;
                }
            });
            document.getElementById("emptyBedsCount").textContent = `عدد الأسرة الفارغة في الطابق الحالي: <br>${roomsWithEmptyBeds.join("<br>")}`;
        }

        function countAllEmptyBeds() {
            let allEmptyBeds = [];
            hotels[currentHotel].floors.forEach((floor, floorIdx) => {
                const emptyBedsInRoom = floor.rooms.filter(room => room.beds.filter(bed => bed === "").length > 0);
                emptyBedsInRoom.forEach(room => {
                    allEmptyBeds.push(`الطابق ${floorIdx + 1} - غرفة ${room.roomNumber}`);
                });
            });
            document.getElementById("allEmptyBedsCount").innerHTML = allEmptyBeds.length ? allEmptyBeds.join("<br>") : "لا يوجد أسرّة فارغة في هذا الفندق.";
        }

        function deleteAllData() {
            if (confirm("هل أنت متأكد أنك تريد حذف جميع البيانات؟")) {
                localStorage.removeItem('hotels');
                alert("تم حذف البيانات.");
                location.reload();
            }
        }

        function searchPerson() {
            const searchTerm = document.getElementById("searchName").value.trim().toLowerCase(); 
            let result = '';
            let found = false;
            for (let floorIdx = 0; floorIdx < hotels[currentHotel].floors.length; floorIdx++) {
                const floor = hotels[currentHotel].floors[floorIdx];
                floor.rooms.forEach((room, roomIdx) => {
                    room.beds.forEach((bed, bedIdx) => {
                        if (bed.toLowerCase().includes(searchTerm)) {
                            result += `الطابق: ${floorIdx + 1}، غرفة: ${room.roomNumber}, سرير: ${bedIdx + 1}, اسم الحاج: ${bed}<br>`;
                            found = true;
                        }
                    });
                });
            }
            document.getElementById("searchResult").innerHTML = found ? result : "لم يتم العثور على الحاج.";
        }
    </script>
</body>
</html>
