<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>eCommerce Campaign Details</title>
    <script src="https://cdn.jsdelivr.net/npm/xlsx@0.16.9/dist/xlsx.full.min.js"></script>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f4f4f4; }
        .editable { background-color: #fff7e6; }
        .notes { width: 100%; height: 50px; }
    </style>
</head>
<body>
    <h2>eCommerce Campaign Details</h2>
    <button onclick="addCampaign()">Add Campaign</button>
    <input type="file" id="uploadExcel" accept=".xlsx" onchange="handleFileUpload(event)">
    <button onclick="saveData()">Save Data</button>
    <table id="campaignTable">
        <thead>
            <tr>
                <th>Campaign Name</th>
                <th>Registration Date</th>
                <th>Template Used</th>
                <th>Notes</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>

    <!-- Firebase SDK -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.4.0/firebase-app.js";
        import { getFirestore, collection, addDoc, getDocs, deleteDoc, doc } from "https://www.gstatic.com/firebasejs/11.4.0/firebase-firestore.js";

        const firebaseConfig = {
            apiKey: "AIzaSyCcqveA7pMuEzvukyA__dQVzKNE-dLf1LM",
            authDomain: "campaign-tracker-6bfc6.firebaseapp.com",
            projectId: "campaign-tracker-6bfc6",
            storageBucket: "campaign-tracker-6bfc6.firebasestorage.app",
            messagingSenderId: "900499590051",
            appId: "1:900499590051:web:f60635007c1f6509eda816",
            measurementId: "G-NB94KVGHGF"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);

        async function addCampaign(campaign = { name: 'New Campaign', date: 'YYYY-MM-DD', template: '', notes: '' }) {
            const table = document.getElementById("campaignTable").getElementsByTagName('tbody')[0];
            let row = table.insertRow();
            row.innerHTML = `
                <td contenteditable="true" class="editable">${campaign.name}</td>
                <td contenteditable="true" class="editable">${campaign.date}</td>
                <td>
                    <input type="file" class="template-upload" accept=".xlsx" onchange="uploadTemplate(this)">
                    ${campaign.template ? `<a href="${campaign.template}" class="download-link" download>Download</a>` : '<a href="#" class="download-link" style="display:none;">Download</a>'}
                </td>
                <td><textarea class="notes">${campaign.notes}</textarea></td>
                <td><button onclick="deleteRow('${campaign.id}', this)">Delete</button></td>
            `;

            if (!campaign.id) {
                const docRef = await addDoc(collection(db, "campaigns"), campaign);
                campaign.id = docRef.id;
                console.log("Campaign added with ID:", docRef.id);
            }
        }

        async function deleteRow(id, button) {
            if (id) {
                await deleteDoc(doc(db, "campaigns", id));
                console.log("Campaign deleted from Firebase");
            }
            let row = button.parentNode.parentNode;
            row.parentNode.removeChild(row);
        }

        async function saveData() {
            let tableRows = document.querySelectorAll("#campaignTable tbody tr");
            tableRows.forEach(async row => {
                let name = row.cells[0].innerText;
                let date = row.cells[1].innerText;
                let template = row.cells[2].querySelector(".download-link").href;
                let notes = row.cells[3].querySelector(".notes").value;

                await addDoc(collection(db, "campaigns"), {
                    name: name,
                    date: date,
                    template: template,
                    notes: notes
                });
                console.log("Campaign saved to Firebase");
            });
        }

        async function loadData() {
            const querySnapshot = await getDocs(collection(db, "campaigns"));
            querySnapshot.forEach(doc => {
                addCampaign({ id: doc.id, ...doc.data() });
            });
        }

        window.onload = loadData;
    </script>
</body>
</html>
