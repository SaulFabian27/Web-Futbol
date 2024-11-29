# Web-Futbol
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestión de Torneo de Fútbol</title>
    <link rel="stylesheet" type="text/css" href="estilos.css">
</head>
<body>
    <!-- Sección de login -->
    <div id="loginSection">
        <h2>Ingresar al Torneo</h2>
        <input type="text" id="loginTeamName" placeholder="Ingresa tu equipo">
        <button onclick="login()">Ingresar</button>
        <p id="loginError" style="color: red; display: none;">El equipo no está registrado</p>
    </div>

    <!-- Sección principal (escondida hasta que el login sea exitoso) -->
    <div id="mainSection" style="display: none;">
        <h1>Gestión de Torneo de Fútbol</h1>

        <!-- Formulario para agregar equipos (solo visible para ti al principio) -->
        <div id="adminSection">
            <h2>Agregar Equipo</h2>
            <input type="text" id="teamName" placeholder="Nombre del equipo">
            <button onclick="addTeam()">Agregar</button>
        </div>

        <h2>Equipos Registrados</h2>
        <ul id="teamList"></ul>

        <!-- Formulario para modificar equipos -->
        <h2>Modificar Equipo</h2>
        <select id="editTeamSelector"></select>
        <input type="text" id="newTeamName" placeholder="Nuevo nombre del equipo">
        <button onclick="editTeam()">Modificar</button>
        <button onclick="removeTeam()">Eliminar</button>

        <!-- Formulario para registrar partidos -->
        <h2>Registrar Partido</h2>
        <select id="homeTeam"></select>
        <select id="awayTeam"></select>
        <input type="number" id="homeScore" placeholder="Goles Local" min="0">
        <input type="number" id="awayScore" placeholder="Goles Visitante" min="0">
        <button onclick="addMatch()">Registrar</button>

        <h2>Partidos Registrados</h2>
        <ul id="matchList"></ul>

        <!-- Tabla de posiciones -->
        <h2>Tabla de Posiciones</h2>
        <table border="1">
            <thead>
                <tr>
                    <th>Equipo</th>
                    <th>Jugados</th>
                    <th>Ganados</th>
                    <th>Empates</th>
                    <th>Perdidos</th>
                    <th>Goles a Favor</th>
                    <th>Goles en Contra</th>
                    <th>Puntos</th>
                </tr>
            </thead>
            <tbody id="standingsTable"></tbody>
        </table>
    </div>

    <script>
        let teams = []; // Aquí se almacenan los equipos registrados
        let matches = [];
        let standings = {};
        let isAdmin = true; // Solo tú eres administrador al principio

        // Al cargar la página, recuperamos los equipos desde localStorage
        window.onload = function() {
            const storedTeams = localStorage.getItem("teams");
            if (storedTeams) {
                teams = JSON.parse(storedTeams);
            }

            if (teams.length === 0) {
                // Si no hay equipos, permitimos acceso directo al administrador
                document.getElementById("loginSection").style.display = "none";
                document.getElementById("mainSection").style.display = "block";
            } else {
                // Si ya hay equipos, mostramos el login
                updateTeamList();
                updateSelectors();
            }
        };

        // Login de usuario
        function login() {
            const teamName = document.getElementById("loginTeamName").value.trim().toLowerCase();  // Eliminar espacios y convertir a minúsculas
            // Verificar si el equipo existe en el sistema
            if (teams.some(team => team.toLowerCase() === teamName)) {  // Ignorar mayúsculas y minúsculas
                // Mostrar la sección principal y ocultar el login
                document.getElementById("loginSection").style.display = "none";
                document.getElementById("mainSection").style.display = "block";
            } else {
                // Mostrar un mensaje de error si el equipo no está registrado
                document.getElementById("loginError").style.display = "block";
            }
        }

        // Agregar un equipo (solo disponible para el administrador)
        function addTeam() {
            if (isAdmin) {
                const teamName = document.getElementById("teamName").value.trim();
                if (teamName && !teams.includes(teamName)) {
                    teams.push(teamName);
                    standings[teamName] = {
                        played: 0,
                        won: 0,
                        drawn: 0,
                        lost: 0,
                        goalsFor: 0,
                        goalsAgainst: 0,
                        points: 0
                    };
                    // Guardar equipos en localStorage
                    localStorage.setItem("teams", JSON.stringify(teams));
                    updateTeamList();
                    updateSelectors();
                    updateStandingsTable();
                }
            }
        }

        // Actualizar lista de equipos
        function updateTeamList() {
            const list = document.getElementById("teamList");
            list.innerHTML = teams.map(team => `<li>${team}</li>`).join('');
            updateEditTeamSelector(); // Actualiza el selector para editar equipos
        }

        // Actualizar selectores
        function updateSelectors() {
            const options = teams.map(team => `<option value="${team}">${team}</option>`).join('');
            document.getElementById("homeTeam").innerHTML = options;
            document.getElementById("awayTeam").innerHTML = options;
        }

        // Actualizar el selector para modificar equipos
        function updateEditTeamSelector() {
            const options = teams.map(team => `<option value="${team}">${team}</option>`).join('');
            document.getElementById("editTeamSelector").innerHTML = options;
        }

        // Modificar un equipo
        function editTeam() {
            const oldName = document.getElementById("editTeamSelector").value;
            const newName = document.getElementById("newTeamName").value.trim();

            if (oldName && newName && !teams.includes(newName)) {
                // Modificar el nombre del equipo en la lista de equipos
                const index = teams.indexOf(oldName);
                teams[index] = newName;
                standings[newName] = standings[oldName]; // Copiar las estadísticas
                delete standings[oldName]; // Eliminar el equipo antiguo
                // Guardar los equipos modificados en localStorage
                localStorage.setItem("teams", JSON.stringify(teams));
                updateTeamList();
                updateSelectors();
                updateStandingsTable();
            }
        }

        // Eliminar un equipo
        function removeTeam() {
            const teamName = document.getElementById("editTeamSelector").value;
            const index = teams.indexOf(teamName);
            if (index !== -1) {
                // Eliminar el equipo de la lista
                teams.splice(index, 1);
                delete standings[teamName]; // Eliminar las estadísticas del equipo
                // Guardar los equipos modificados en localStorage
                localStorage.setItem("teams", JSON.stringify(teams));
                updateTeamList();
                updateSelectors();
                updateStandingsTable();
            }
        }

        // Registrar un partido
        function addMatch() {
            const homeTeam = document.getElementById("homeTeam").value;
            const awayTeam = document.getElementById("awayTeam").value;
            const homeScore = parseInt(document.getElementById("homeScore").value);
            const awayScore = parseInt(document.getElementById("awayScore").value);

            if (homeTeam && awayTeam && homeScore >= 0 && awayScore >= 0 && homeTeam !== awayTeam) {
                matches.push({ homeTeam, awayTeam, homeScore, awayScore });
                updateMatchList();
                updateStandings(homeTeam, awayTeam, homeScore, awayScore);
                updateStandingsTable();
            }
        }

        // Actualizar lista de partidos
        function updateMatchList() {
            const list = document.getElementById("matchList");
            list.innerHTML = matches.map((match, index) =>
                `<li>${match.homeTeam} (${match.homeScore}) vs (${match.awayScore}) ${match.awayTeam}
                    <button onclick="removeMatch(${index})">Eliminar</button>
                </li>`
            ).join('');
        }

        // Actualizar la tabla de posiciones
        function updateStandings(homeTeam, awayTeam, homeScore, awayScore) {
            standings[homeTeam].played++;
            standings[awayTeam].played++;

            standings[homeTeam].goalsFor += homeScore;
            standings[awayTeam].goalsFor += awayScore;
            standings[homeTeam].goalsAgainst += awayScore;
            standings[awayTeam].goalsAgainst += homeScore;

            if (homeScore > awayScore) {
                standings[homeTeam].won++;
                standings[awayTeam].lost++;
                standings[homeTeam].points += 3;
            } else if (homeScore < awayScore) {
                standings[awayTeam].won++;
                standings[homeTeam].lost++;
                standings[awayTeam].points += 3;
            } else {
                standings[homeTeam].drawn++;
                standings[awayTeam].drawn++;
                standings[homeTeam].points++;
                standings[awayTeam].points++;
            }
        }

        // Mostrar la tabla de posiciones
        function updateStandingsTable() {
            const tableBody = document.getElementById("standingsTable");
            tableBody.innerHTML = teams.map(team => {
                const s = standings[team];
                return `
                    <tr>
                        <td>${team}</td>
                        <td>${s.played}</td>
                        <td>${s.won}</td>
                        <td>${s.drawn}</td>
                        <td>${s.lost}</td>
                        <td>${s.goalsFor}</td>
                        <td>${s.goalsAgainst}</td>
                        <td>${s.points}</td>
                    </tr>
                `;
            }).join('');
        }
    </script>
</body>
</html>
