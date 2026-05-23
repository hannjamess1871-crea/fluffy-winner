// Game state
const gameState = {
    player: {
        x: 500,
        y: 300,
        width: 30,
        height: 40,
        speed: 3,
        money: 1000,
        health: 100,
        hunger: 100,
        energy: 100,
        inventory: [],
        job: null,
        inBuilding: null
    },
    world: {
        time: 12 * 60, // minutes from midnight
        day: 1,
        weather: 'sunny'
    },
    buildings: [],
    npcs: [],
    items: [],
    keys: {}
};

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

// Initialize buildings
function initBuildings() {
    gameState.buildings = [
        { x: 100, y: 100, width: 120, height: 100, type: 'house', name: 'Home', color: '#ff6b6b' },
        { x: 300, y: 100, width: 120, height: 100, type: 'store', name: 'Store', color: '#4ecdc4' },
        { x: 500, y: 100, width: 120, height: 100, type: 'gym', name: 'Gym', color: '#ffe66d' },
        { x: 700, y: 100, width: 120, height: 100, type: 'restaurant', name: 'Restaurant', color: '#95e1d3' },
        { x: 100, y: 350, width: 120, height: 100, type: 'office', name: 'Office', color: '#a8e6cf' },
        { x: 300, y: 350, width: 120, height: 100, type: 'park', name: 'Park', color: '#90ee90' },
        { x: 500, y: 350, width: 120, height: 100, type: 'hospital', name: 'Hospital', color: '#ffcccc' },
        { x: 700, y: 350, width: 120, height: 100, type: 'casino', name: 'Casino', color: '#dda0dd' }
    ];
}

// Initialize NPCs
function initNPCs() {
    gameState.npcs = [
        { x: 150, y: 150, name: 'Alice', dialogue: 'Hi! Want to chat?' },
        { x: 350, y: 150, name: 'Bob', dialogue: 'Check out my store!' },
        { x: 550, y: 150, name: 'Coach', dialogue: 'Time to get fit!' },
        { x: 750, y: 150, name: 'Chef', dialogue: 'Hungry? Come in!' }
    ];
}

// Key event listeners
document.addEventListener('keydown', (e) => {
    gameState.keys[e.key.toLowerCase()] = true;
});

document.addEventListener('keyup', (e) => {
    gameState.keys[e.key.toLowerCase()] = false;
});

// Mouse click for interactions
canvas.addEventListener('click', (e) => {
    const rect = canvas.getBoundingClientRect();
    const mouseX = e.clientX - rect.left;
    const mouseY = e.clientY - rect.top;

    // Check if clicked on building
    gameState.buildings.forEach(building => {
        if (mouseX >= building.x && mouseX <= building.x + building.width &&
            mouseY >= building.y && mouseY <= building.y + building.height) {
            enterBuilding(building);
        }
    });

    // Check if clicked on NPC
    gameState.npcs.forEach(npc => {
        const dist = Math.hypot(npc.x - mouseX, npc.y - mouseY);
        if (dist < 40) {
            showInfo(npc.dialogue);
        }
    });
});

// Enter building
function enterBuilding(building) {
    gameState.player.inBuilding = building.type;
    showInfo(`You entered the ${building.name}. (Press ESC to exit)`);
}

// Exit building
function exitBuilding() {
    gameState.player.inBuilding = null;
}

// Show info
function showInfo(text) {
    document.getElementById('infoText').textContent = text;
}

// Handle building interactions
function handleBuildingAction(buildingType) {
    switch(buildingType) {
        case 'store':
            gameState.player.money -= 50;
            gameState.player.hunger += 20;
            gameState.player.inventory.push('Food');
            showInfo('You bought food for $50!');
            break;
        case 'gym':
            gameState.player.energy -= 30;
            gameState.player.health += 10;
            showInfo('You worked out! Health improved.');
            break;
        case 'restaurant':
            gameState.player.money -= 30;
            gameState.player.hunger += 30;
            showInfo('You had a nice meal!');
            break;
        case 'office':
            gameState.player.money += 100;
            gameState.player.energy -= 40;
            showInfo('You worked and earned $100!');
            break;
        case 'hospital':
            gameState.player.money -= 100;
            gameState.player.health = 100;
            showInfo('You got healed! Cost: $100');
            break;
        case 'house':
            gameState.player.energy = 100;
            gameState.player.hunger = 100;
            showInfo('You rested at home.');
            break;
        case 'park':
            gameState.player.energy += 20;
            gameState.player.health += 5;
            showInfo('You enjoyed the park!');
            break;
        case 'casino':
            const win = Math.random() > 0.5;
            const amount = Math.floor(Math.random() * 500) + 50;
            gameState.player.money += win ? amount : -amount;
            showInfo(win ? `You won $${amount}!` : `You lost $${amount}...`);
            break;
    }
}

// Update player movement
function updatePlayer() {
    if (gameState.keys['w'] || gameState.keys['arrowup']) gameState.player.y -= gameState.player.speed;
    if (gameState.keys['s'] || gameState.keys['arrowdown']) gameState.player.y += gameState.player.speed;
    if (gameState.keys['a'] || gameState.keys['arrowleft']) gameState.player.x -= gameState.player.speed;
    if (gameState.keys['d'] || gameState.keys['arrowright']) gameState.player.x += gameState.player.speed;

    // Boundary checking
    gameState.player.x = Math.max(0, Math.min(canvas.width - gameState.player.width, gameState.player.x));
    gameState.player.y = Math.max(0, Math.min(canvas.height - gameState.player.height, gameState.player.y));

    // Check for building collision
    gameState.buildings.forEach(building => {
        if (gameState.player.x < building.x + building.width &&
            gameState.player.x + gameState.player.width > building.x &&
            gameState.player.y < building.y + building.height &&
            gameState.player.y + gameState.player.height > building.y) {
            gameState.player.inBuilding = building.type;
        }
    });
}

// Update game time and stats
function updateGameState() {
    gameState.world.time += 0.5;
    
    if (gameState.world.time >= 1440) {
        gameState.world.time = 0;
        gameState.world.day++;
    }

    // Stats decay
    gameState.player.hunger = Math.max(0, gameState.player.hunger - 0.1);
    gameState.player.energy = Math.max(0, gameState.player.energy - 0.05);
    
    if (gameState.player.hunger < 30) {
        gameState.player.health = Math.max(0, gameState.player.health - 0.2);
    }

    if (gameState.player.energy < 20) {
        gameState.player.health = Math.max(0, gameState.player.health - 0.1);
    }
}

// Draw function
function draw() {
    // Clear canvas
    ctx.fillStyle = '#2a2a2a';
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Draw grid
    ctx.strokeStyle = '#444';
    ctx.lineWidth = 0.5;
    for (let i = 0; i < canvas.width; i += 50) {
        ctx.beginPath();
        ctx.moveTo(i, 0);
        ctx.lineTo(i, canvas.height);
        ctx.stroke();
    }
    for (let i = 0; i < canvas.height; i += 50) {
        ctx.beginPath();
        ctx.moveTo(0, i);
        ctx.lineTo(canvas.width, i);
        ctx.stroke();
    }

    // Draw buildings
    gameState.buildings.forEach(building => {
        ctx.fillStyle = building.color;
        ctx.fillRect(building.x, building.y, building.width, building.height);
        ctx.strokeStyle = '#fff';
        ctx.lineWidth = 2;
        ctx.strokeRect(building.x, building.y, building.width, building.height);

        // Draw building name
        ctx.fillStyle = '#fff';
        ctx.font = 'bold 12px Arial';
        ctx.textAlign = 'center';
        ctx.fillText(building.name, building.x + building.width / 2, building.y + building.height / 2);
    });

    // Draw NPCs
    gameState.npcs.forEach(npc => {
        ctx.fillStyle = '#ff00ff';
        ctx.beginPath();
        ctx.arc(npc.x, npc.y, 12, 0, Math.PI * 2);
        ctx.fill();
        ctx.fillStyle = '#fff';
        ctx.font = 'bold 10px Arial';
        ctx.textAlign = 'center';
        ctx.fillText(npc.name, npc.x, npc.y + 25);
    });

    // Draw player
    ctx.fillStyle = '#00ff00';
    ctx.fillRect(gameState.player.x, gameState.player.y, gameState.player.width, gameState.player.height);
    ctx.strokeStyle = '#fff';
    ctx.lineWidth = 2;
    ctx.strokeRect(gameState.player.x, gameState.player.y, gameState.player.width, gameState.player.height);

    // Draw player eyes
    ctx.fillStyle = '#000';
    ctx.beginPath();
    ctx.arc(gameState.player.x + 10, gameState.player.y + 10, 3, 0, Math.PI * 2);
    ctx.fill();
    ctx.beginPath();
    ctx.arc(gameState.player.x + 20, gameState.player.y + 10, 3, 0, Math.PI * 2);
    ctx.fill();
}

// Update HUD
function updateHUD() {
    document.getElementById('money').textContent = Math.floor(gameState.player.money);
    document.getElementById('health').textContent = Math.floor(gameState.player.health);
    document.getElementById('hunger').textContent = Math.floor(gameState.player.hunger);
    document.getElementById('energy').textContent = Math.floor(gameState.player.energy);

    const hours = Math.floor(gameState.world.time / 60);
    const minutes = Math.floor(gameState.world.time % 60);
    document.getElementById('time').textContent = 
        `${String(hours).padStart(2, '0')}:${String(minutes).padStart(2, '0')}`;
    document.getElementById('day').textContent = `Day ${gameState.world.day}`;

    // Update actions
    const actionsList = document.getElementById('actionsList');
    actionsList.innerHTML = '';

    if (gameState.player.inBuilding) {
        const btn = document.createElement('button');
        btn.className = 'action-btn';
        btn.textContent = `Use ${gameState.player.inBuilding}`;
        btn.onclick = () => handleBuildingAction(gameState.player.inBuilding);
        actionsList.appendChild(btn);

        const exitBtn = document.createElement('button');
        exitBtn.className = 'action-btn';
        exitBtn.textContent = 'Exit';
        exitBtn.onclick = () => exitBuilding();
        actionsList.appendChild(exitBtn);
    } else {
        const restBtn = document.createElement('button');
        restBtn.className = 'action-btn';
        restBtn.textContent = 'Rest (E)';
        restBtn.onclick = () => {
            gameState.player.energy = 100;
            showInfo('You rested.');
        };
        actionsList.appendChild(restBtn);
    }

    // Update inventory
    const inventoryList = document.getElementById('inventoryList');
    inventoryList.innerHTML = '';
    if (gameState.player.inventory.length === 0) {
        inventoryList.innerHTML = '<span style="color: #888;">Empty</span>';
    } else {
        gameState.player.inventory.forEach((item, idx) => {
            const itemDiv = document.createElement('div');
            itemDiv.className = 'inventory-item';
            itemDiv.textContent = item;
            itemDiv.onclick = () => {
                gameState.player.inventory.splice(idx, 1);
                gameState.player.hunger = Math.min(100, gameState.player.hunger + 20);
                showInfo(`You used ${item}`);
            };
            inventoryList.appendChild(itemDiv);
        });
    }
}

// Game loop
function gameLoop() {
    updatePlayer();
    updateGameState();
    draw();
    updateHUD();
    requestAnimationFrame(gameLoop);
}

// Initialize and start
initBuildings();
initNPCs();
gameLoop();

showInfo('Welcome to the Modern World Sandbox! Use arrow keys/WASD to move. Click on buildings to enter. Click on NPCs to talk.');
