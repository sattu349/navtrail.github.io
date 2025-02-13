<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Trip Tailor - Travel Planner</title>
    <link rel="stylesheet" href="styles.css">
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-firestore.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-auth.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
            color: #333;
            transition: background-color 0.3s, color 0.3s;
        }
        .dark-mode {
            background-color: #2c3e50;
            color: #ecf0f1;
        }
        header {
            background: #2c3e50;
            color: white;
            padding: 15px;
            text-align: center;
            position: sticky;
            top: 0;
            z-index: 1000;
        }
        nav ul {
            list-style: none;
            padding: 0;
            display: flex;
            justify-content: center;
        }
        nav ul li {
            margin: 0 15px;
        }
        nav ul li a {
            color: white;
            text-decoration: none;
            font-weight: bold;
        }
        section {
            padding: 20px;
            text-align: center;
            opacity: 0;
            transform: translateY(20px);
            animation: fadeInUp 0.6s forwards;
        }
        @keyframes fadeInUp {
            from {
                opacity: 0;
                transform: translateY(20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
        .destination-card:hover {
            transform: scale(1.05);
            transition: transform 0.3s;
        }
        button {
            background: #3498db;
            color: white;
            border: none;
            padding: 10px 15px;
            cursor: pointer;
            border-radius: 5px;
        }
        button:hover {
            background: #2980b9;
        }
        #loading {
            display: none;
            font-size: 16px;
            color: #3498db;
        }
    </style>
</head>
<body>
    <header>
        <h1>Trip Tailor</h1>
        <nav>
            <ul>
                <li><a href="#home">Home</a></li>
                <li><a href="#destinations">Destinations</a></li>
                <li><a href="#planner">AI Itinerary Planner</a></li>
                <li><button onclick="toggleDarkMode()">Toggle Dark Mode</button></li>
                <li><button onclick="signIn()">Sign In</button></li>
            </ul>
        </nav>
    </header>
    
    <section id="home">
        <h2>Welcome to Trip Tailor</h2>
        <p>Plan your perfect trip with AI-powered itineraries and hidden gems.</p>
    </section>
    
    <section id="destinations">
        <h2>Popular Destinations</h2>
        <div id="destination-list"></div>
    </section>
    
    <section id="planner">
        <h2>AI Itinerary Planner</h2>
        <label for="destination">Choose a Destination:</label>
        <select id="destination"></select>
        <button onclick="generateItinerary()">Plan My Trip</button>
        <div id="loading">Generating itinerary...</div>
        <div id="itinerary"></div>
    </section>

    <script>
        const firebaseConfig = {
            apiKey: "your-api-key",
            authDomain: "your-auth-domain",
            projectId: "your-project-id",
            storageBucket: "your-storage-bucket",
            messagingSenderId: "your-messaging-sender-id",
            appId: "your-app-id"
        };

        firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();
        const auth = firebase.auth();

        function toggleDarkMode() {
            document.body.classList.toggle("dark-mode");
        }

        function signIn() {
            const provider = new firebase.auth.GoogleAuthProvider();
            auth.signInWithPopup(provider)
                .then(result => {
                    alert(`Welcome, ${result.user.displayName}`);
                })
                .catch(error => {
                    console.error("Error signing in:", error);
                });
        }

        function loadDestinations() {
            let destinationList = document.getElementById("destination-list");
            let destinationDropdown = document.getElementById("destination");
            
            db.collection("destinations").where("popular", "==", true).get()
                .then(querySnapshot => {
                    querySnapshot.forEach(doc => {
                        let data = doc.data();
                        let card = document.createElement("div");
                        card.className = "destination-card";
                        card.textContent = data.name;
                        destinationList.appendChild(card);

                        let option = document.createElement("option");
                        option.value = doc.id;
                        option.textContent = data.name;
                        destinationDropdown.appendChild(option);
                    });
                })
                .catch(error => {
                    console.error("Error loading destinations:", error);
                });
        }

        async function generateItinerary() {
            const destination = document.getElementById("destination").value;
            document.getElementById("loading").style.display = "block";
            
            try {
                const response = await fetch("https://your-ai-backend.com/generate-itinerary", {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({ destination: destination })
                });
                
                if (!response.ok) {
                    throw new Error("Failed to fetch itinerary");
                }
                
                const data = await response.json();
                document.getElementById("itinerary").innerHTML = `<h3>Your AI-Generated Itinerary:</h3><p>${data.itinerary}</p>`;
            } catch (error) {
                document.getElementById("itinerary").innerHTML = "Error generating itinerary. Please try again.";
                console.error("Error:", error);
            } finally {
                document.getElementById("loading").style.display = "none";
            }
        }

        document.addEventListener("DOMContentLoaded", loadDestinations);
    </script>
</body>
</html>

