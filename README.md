# Smart in the Dark — a Serious Game for Arduino 

**Smart in the Dark** is an educational serious game built using **GDevelop** and **BlocklyDuino**. It simulates Arduino-based challenges through narrative-driven levels, with an embedded block-based code editor that validates logic and enhances learning.

---

##  Architecture Overview

**Smart in the Dark** follows a modular, event-driven system architecture composed of the following main components:

### 1.  GDevelop Frontend 

- Built using **GDevelop 5**, the game logic is structured in scenes and events.
- It handles:
  - Level interactions and story progression
  - Embedding BlocklyDuino via an `<iframe>`
  - Receiving and validating code
  - Sending immediate feedback to players

### 2. BlocklyDuino Editor 1.4 (Embedded IFrame)

- A fork of [BlocklyDuino](https://github.com/HATEMMerabtine/blocklyduino-serious-game), redesigned for integration into GDevelop.
- Each level has a unique URL (`/level/1`, `/level/2`, etc.) with:
  - Level-specific blocks and toolbox
  - Preloaded workspace templates
  - Level-specific documentation/help
- Sends Arduino-like code and click data to GDevelop via `window.postMessage()`

### 3.  Firebase 

- Used to **track player behavior**, such as:
  - Number of interface interactions (clicks)
  -Level Completion Time

This term clearly conveys that you are measuring the amount of time each player spends to complete a given level. It’s widely understood in game analytics and serious games research.

You can also define it briefly, for example:

“Level Completion Time is defined as the total time (in seconds ) that a player spends to complete a specific level in the game, as recorded via Firebase.”
- Firebase Authentication can be added to support:
  - User login (email/Google)
- Firebase Firestore/Realtime DB can be configured from inside GDevelop ([GDevelop Firebase Docs](https://wiki.gdevelop.io/gdevelop5/all-features/firebase/quickstart/))

### 4. JavaScript Bridge (Inter-Component Messaging)

- GDevelop uses `window.addEventListener('message', handler)` to receive:
  - `blocklyCode` — the generated Arduino code string
  - `clickCount` — the number of user interactions
- The game engine sends messages **back** to BlocklyDuino for visual feedback (`Correct!`, `Try Again!`) via `iframe.contentWindow.postMessage()`[read more details](https://github.com/HATEMMerabtine/blocklyduino-serious-game).

---

##  Core Features

- Blockly code editor with drag-and-drop functionality embedded through an iframe. 
- Real-time feedback loop via JavaScript `postMessage`  
- Level-based toolboxes and guided logic  
- track player behavior withe Firebase  

---

##  Technologies Used
GDevelop offers multiple export options to accommodate different deployment needs, including native exports for Windows, macOS, Linux, Android, and iOS, as well as an HTML5 export for web-based games. For our project, we chose to export the game in HTML5 format. This choice was motivated by the ease of user access—players can run the game directly in any modern web browser without needing to install additional software—and by the simplified data collection process for evaluation purposes, as web technologies facilitate tracking user interactions and performance remotely and in real time.
This approach ensures seamless integration with educational platforms and supports remote performance monitoring.

| Component     | Tool / Framework                              | Purpose                             |
|---------------|-----------------------------------------------|-------------------------------------|
| Game Engine   | GDevelop 5.5.231                              | Game scenes and logic               |
| Code Editor   | BlocklyDuino (custom based on 1.4)            | Visual block programming (iframe)   |
| Hosting       | Netlify                                       | Static deployment                   |
| JS Bridge     | `window.postMessage` API                      | Inter-component communication       |
|     DB        | Firebase Auth + Firestore                     | User tracking                       |

---


##  Local Development Setup

###  Requirements

| Tool             | Purpose                                      |
|------------------|----------------------------------------------|
| **GDevelop 5**   | To open and edit the source code             |
| **Web server**   | To host the exported HTML5 version (optional)|
| **Web browser**  | To run and test the game                     |

---

### Steps

1. **Open the repository**  
    [https://github.com/HATEMMerabtine/smart-in-the-dark-serious-game](https://github.com/HATEMMerabtine/smart-in-the-dark-serious-game)

2. **Download the project folder**  
   - Click the green **Code** button  
   - Choose **Download ZIP**  
   - Extract the folder

3. **Open the project in GDevelop 5**  
   - Launch GDevelop 5  
   - Select **Open a project**, and choose the extracted folder  
   - Access scenes, logic, variables, and assets

4. **Modify as needed**  
   ####  Add New Levels

To create new levels, use multiple scenes for narrative and programming tasks.

#####  Multi-Scene Level Design (Recommended)

| Scene Name         | Purpose                                                                 |
|--------------------|-------------------------------------------------------------------------|
| `Level3_intro`     | Introduce the problem (e.g., a room with no light)                      |
| `Level3_blockly`   | Embed BlocklyDuino to solve the task                                    |
| `Level3_simulator` | presents  the simulator with specific components for the level          |
| `Level3_result`    | Reflect on outcome and summarize learning                               |

---

####  Steps for Building a New Level

1. **Scene: `Level3_intro`**
   - Create visual context (broken light, dark room, etc.)
   - Add narration/dialogue
   - Add a button → Change scene to `Level3_blockly`

2. **Scene: `Level3_blockly`**

 a) **Embed the Blockly iframe**

```javascript
const iframe = document.createElement('iframe');
iframe.src = "https://blocklyduino-smart-in-the-dark-sg.netlify.app/level/3";
iframe.id = "blocklyIframe";
iframe.style.position = "absolute";
iframe.style.top = "100px";
iframe.style.left = "5%";
iframe.style.width = "90%";
iframe.style.height = "500px";
iframe.style.border = "none";
iframe.style.zIndex = "999";
document.body.appendChild(iframe);
```

 b) **Receive messages from BlocklyDuino**

```javascript
// --- LEVEL 3: SETUP UNIFIED LISTENER ---

console.log(" GDevelop Level 3: Initializing listeners...");

// Define ONE handler function for ALL Level 3 messages
window.handleLevel3Messages = function(event) {
  if (!event.data || !event.data.type) return; // Ignore invalid messages

  const sceneVariables = runtimeScene.getVariables();
  const iframe = document.getElementById("myIframe");

  // --- Handle Blockly Code ---
  if (event.data.type === "blocklyCode") {
    let blocklyCode = event.data.code.replace(/\s+/g, '');
    console.log(" L3 Received code:", blocklyCode);

    // Store code in a unique scene variable for this level
    sceneVariables.get("BlocklyCode3").setString(blocklyCode);
    console.log(" Stored code in 'BlocklyCode3'.");
    
    // Validate the code
    const correctCode = sceneVariables.get("correct").getAsString();
    let feedbackMessage = (correctCode === blocklyCode) ? "Correct!" : "Incorrect, try again!";
    
    // Send feedback to iframe
    if (iframe && iframe.contentWindow) {
      iframe.contentWindow.postMessage({ type: "blocklyFeedback", message: feedbackMessage }, "*");
    }
  }

  // --- Handle Click Count ---
  if (event.data.type === "clickCount") {
    let clickCount = event.data.clicks;
    console.log(" L3 Received clicks:", clickCount);
    // Store in a unique scene variable for this level
    sceneVariables.get("ClickCount3").setNumber(clickCount);
    console.log("Stored clicks in 'ClickCount3'.");
  }
};

// Add the listener using its unique flag
if (!window.level3ListenerActive) {
  window.addEventListener("message", window.handleLevel3Messages, false);
  window.level3ListenerActive = true;
  console.log(" Level 3 Unified Listener ADDED.");
}
```

 c) **Validate the player's code**

- Compare `Variable(BlocklyCode3)` to the correct solution.
- If correct:
wite data to firebase.
delete iframe .
remove the eventListener :
```javascript
// --- LEVEL 3: CLEANUP LISTENER ---

if (window.level3ListenerActive) {
  window.removeEventListener("message", window.handleLevel3Messages, false);
  window.level3ListenerActive = false;
  console.log(" Level 3 listener REMOVED.");
}

```
   Change scene to `Level3_simulator`

3. **Scene: `Level3_simulator`**
   - Create work space or vertual arduino lab (LEDs , Arduino board , etc.)
   - Add help/hints box 
   - use event system to permet circuit manipulation 

4. **Scene: `Level3_result`**
After success, guide the player to a result/reflection scene:

- Recap the problem and how the code fixed it
- Provide feedback and motivation
---


5. **Export as HTML5**  
   - Go to **File > Export > Web (HTML5)**  
   - Export to a folder (e.g., `gdevelop-export/`)

6. **Host the exported game**  
   - On a local server (Apache, Nginx, Python HTTP)  
   - Or use online hosts like [Netlify](https://netlify.com), or GitHub Pages.

7. **Run the game**  
   - Open `index.html` in a browser  
   - Your updated game is live!

---

## BlocklyDuino Integration

[See full integration method here →](https://github.com/HATEMMerabtine/blocklyduino-serious-game/blob/main/README.md)

---

##  Contributors

- [**Hatem Merabtine**](https://github.com/HATEMMerabtine)  
- [**Dr. Rida Mezghache**](https://github.com/Rida-Mezghache)

---

##  License

MIT License
